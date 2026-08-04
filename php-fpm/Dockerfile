# syntax=docker/dockerfile:1

FROM --platform=$BUILDPLATFORM alpine:3.24 AS ffmpeg-downloader

ARG TARGETARCH
ARG FFMPEG_CHECKSUMS_SHA256

WORKDIR /tmp
# 下载工具刻意跟随 Alpine 3.24 的当前补丁版本，且不会进入最终镜像。
# hadolint ignore=DL3018
RUN --mount=type=tmpfs,target=/tmp set -eux; \
    apk add --no-cache --virtual .download-deps ca-certificates curl xz; \
    case "$TARGETARCH" in \
        amd64) ffmpeg_arch="linux64" ;; \
        arm64) ffmpeg_arch="linuxarm64" ;; \
        *) echo "Unsupported architecture: $TARGETARCH" >&2; exit 1 ;; \
    esac; \
    asset="ffmpeg-master-latest-${ffmpeg_arch}-gpl-shared.tar.xz"; \
    release_url="https://github.com/BtbN/FFmpeg-Builds/releases/download/latest"; \
    curl -fL --retry 5 --retry-all-errors -o "/tmp/$asset" "$release_url/$asset"; \
    curl -fL --retry 5 --retry-all-errors -o /tmp/checksums.sha256 "$release_url/checksums.sha256"; \
    if [ -n "$FFMPEG_CHECKSUMS_SHA256" ]; then \
        echo "$FFMPEG_CHECKSUMS_SHA256  /tmp/checksums.sha256" > /tmp/checksums.sha256.expected; \
        sha256sum -c /tmp/checksums.sha256.expected; \
    fi; \
    grep "  $asset\$" /tmp/checksums.sha256 > /tmp/ffmpeg.sha256; \
    sha256sum -c ffmpeg.sha256; \
    mkdir -p /tmp/ffmpeg /opt/ffmpeg/bin /opt/ffmpeg/lib /opt/ffmpeg/licenses; \
    tar -xJf "$asset" -C /tmp/ffmpeg; \
    archive_root="/tmp/ffmpeg/${asset%.tar.xz}"; \
    install -m 0755 "$archive_root/bin/ffmpeg" /opt/ffmpeg/bin/ffmpeg; \
    install -m 0755 "$archive_root/bin/ffprobe" /opt/ffmpeg/bin/ffprobe; \
    cp -a "$archive_root/lib/." /opt/ffmpeg/lib/; \
    install -m 0644 "$archive_root/LICENSE.txt" /opt/ffmpeg/licenses/LICENSE.txt; \
    chown -R 0:0 /opt/ffmpeg; \
    apk del .download-deps

FROM php:fpm

ARG REDIS_VERSION
ARG IMAGICK_VERSION

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# 安装运行工具并构建 PHP 扩展；构建完成后只保留扩展所需的运行库。
# apt 包刻意跟随当前基础镜像的软件源，以满足自动更新策略。
# hadolint ignore=DL3008
RUN set -eux; \
    mapfile -t saved_apt_mark < <(apt-mark showmanual); \
    apt-get update; \
    apt-get install -y --no-install-recommends \
        git \
        unzip \
        libfreetype-dev \
        libjpeg62-turbo-dev \
        libpng-dev \
        libzip-dev \
        libmagickwand-dev; \
    redis_package="redis${REDIS_VERSION:+-$REDIS_VERSION}"; \
    imagick_package="imagick${IMAGICK_VERSION:+-$IMAGICK_VERSION}"; \
    MAKEFLAGS="-j$(nproc)" pecl install "$redis_package" "$imagick_package"; \
    docker-php-ext-configure gd --with-freetype --with-jpeg; \
    docker-php-ext-install -j"$(nproc)" gd zip pcntl pdo pdo_mysql; \
    docker-php-ext-enable redis imagick; \
    pecl clear-cache; \
    rm -rf /tmp/pear; \
    apt-mark auto '.*' > /dev/null; \
    apt-mark manual "${saved_apt_mark[@]}" git unzip; \
    declare -A runtime_package_set=(); \
    while IFS= read -r library; do \
        resolved_library="$(readlink -f "$library")"; \
        package="$( \
            dpkg-query --search "$resolved_library" \
                | awk -F': ' '$1 !~ /^diversion / { print $1; exit }' \
        )"; \
        if [ -z "$package" ]; then \
            echo "Unable to map runtime library to a package: $library" >&2; \
            exit 1; \
        fi; \
        runtime_package_set["$package"]=1; \
    done < <( \
        find /usr/local -type f \( -perm -0100 -o -name '*.so' \) -exec ldd '{}' ';' 2>/dev/null \
            | awk '/=>/ && $(NF-1) !~ /^\/usr\/local\// { print $(NF-1) }' \
            | sort -u \
    ); \
    runtime_packages=("${!runtime_package_set[@]}"); \
    if (( ${#runtime_packages[@]} )); then apt-mark manual "${runtime_packages[@]}"; fi; \
    apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
    rm -rf /var/lib/apt/lists/*

COPY --from=ffmpeg-downloader /opt/ffmpeg/bin/ /usr/local/bin/
COPY --from=ffmpeg-downloader /opt/ffmpeg/lib/ /usr/local/lib/ffmpeg/
COPY --from=ffmpeg-downloader /opt/ffmpeg/licenses/ /usr/share/licenses/ffmpeg/
COPY --from=composer/composer:latest-bin /composer /usr/bin/composer

COPY custom.ini /usr/local/etc/php/conf.d/custom.ini
COPY zz-docker.conf /usr/local/etc/php-fpm.d/zz-docker.conf

# 构建期验证；任一命令失败都会中止构建。
# hadolint ignore=SC2016
RUN set -eux; \
    echo '/usr/local/lib/ffmpeg' > /etc/ld.so.conf.d/ffmpeg.conf; \
    ldconfig; \
    php --version; \
    php-fpm -tt; \
    composer --version; \
    ffmpeg -version; \
    ffprobe -version; \
    php -r '$required = ["gd", "imagick", "pcntl", "pdo", "pdo_mysql", "redis", "zip"]; foreach ($required as $extension) { if (!extension_loaded($extension)) { fwrite(STDERR, "Missing extension: $extension\n"); exit(1); } }'

SHELL ["/bin/sh", "-c"]
