FROM php:fpm

# 安装 php 扩展
RUN apt-get update \
    && apt-get install -y \
        libfreetype-dev \
        libjpeg62-turbo-dev \
        libpng-dev \
        libzip-dev \
	libmagickwand-dev \
    && rm -rf /var/lib/apt/lists/* \
    && pecl install redis \
    	&& pecl install imagick \
	&& docker-php-ext-configure gd --with-freetype --with-jpeg \
	&& docker-php-ext-install -j$(nproc) gd zip pcntl pdo pdo_mysql \
	&& docker-php-ext-enable redis \
 	&& docker-php-ext-enable imagick

# 设置架构变量
ARG TARGETARCH

# 安装 FFmpeg 7（兼容 amd64/arm64）
RUN set -ex; \
    \
    # 检测架构并设置下载URL
    case "${TARGETARCH}" in \
        amd64) \
            FFMPEG_URL="https://johnvansickle.com/ffmpeg/releases/ffmpeg-release-amd64-static.tar.xz" \
            ;; \
        arm64) \
            FFMPEG_URL="https://johnvansickle.com/ffmpeg/releases/ffmpeg-release-arm64-static.tar.xz" \
            ;; \
        *) echo "Unsupported architecture: ${TARGETARCH}"; exit 1 ;; \
    esac; \
    \
    # 安装依赖工具
    apt-get update && apt-get install -y \
        wget \
        xz-utils \
        && rm -rf /var/lib/apt/lists/*; \
    \
    # 下载并安装 FFmpeg
    wget -q "${FFMPEG_URL}" -O ffmpeg.tar.xz; \
    tar xf ffmpeg.tar.xz; \
    rm ffmpeg.tar.xz; \
    mv ffmpeg-*-static/ffmpeg /usr/local/bin/; \
    mv ffmpeg-*-static/ffprobe /usr/local/bin/; \
    rm -rf ffmpeg-*-static; \
    \
    # 设置权限
    chmod +x /usr/local/bin/ffmpeg /usr/local/bin/ffprobe

# 验证安装
RUN ffmpeg -version | head -n1

RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
    && php composer-setup.php \
    && php -r "unlink('composer-setup.php');" \
    && mv composer.phar /usr/local/bin/composer

# disable log
RUN echo "access.log = /dev/null" >> /usr/local/etc/php-fpm.d/www.conf

# max_children
RUN echo 'pm.max_children = 100' >> /usr/local/etc/php-fpm.d/zz-docker.conf && \
    echo 'pm.max_requests = 500' >> /usr/local/etc/php-fpm.d/zz-docker.conf
