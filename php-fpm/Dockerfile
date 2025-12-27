FROM php:fpm

# 一次性安装所有依赖和扩展
RUN apt-get update && apt-get install -y --no-install-recommends \
        git \
        unzip \
        libfreetype-dev \
        libjpeg62-turbo-dev \
        libpng-dev \
        libzip-dev \
        libmagickwand-dev \
        wget \
        xz-utils \
    && pecl install redis imagick \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) gd zip pcntl pdo pdo_mysql \
    && docker-php-ext-enable redis imagick \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# 设置架构变量
ARG TARGETARCH

# 安装FFmpeg（兼容amd64/arm64）
RUN case "${TARGETARCH}" in \
        amd64) \
            FFMPEG_URL="https://github.com/BtbN/FFmpeg-Builds/releases/download/latest/ffmpeg-master-latest-linux64-gpl.tar.xz" \
            ;; \
        arm64) \
            FFMPEG_URL="https://github.com/BtbN/FFmpeg-Builds/releases/download/latest/ffmpeg-master-latest-linuxarm64-gpl.tar.xz" \
            ;; \
        *) echo "Unsupported architecture: ${TARGETARCH}"; exit 1 ;; \
    esac \
    && wget -q "${FFMPEG_URL}" -O ffmpeg.tar.xz \
    && tar -xf ffmpeg.tar.xz --strip-components=1 \
    && mv bin/ffmpeg bin/ffprobe /usr/local/bin/ \
    && chmod +x /usr/local/bin/ffmpeg /usr/local/bin/ffprobe \
    && rm -rf ffmpeg* *.tar.xz

# 安装Composer
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
    && php composer-setup.php --install-dir=/usr/local/bin --filename=composer \
    && rm composer-setup.php
    && mkdir -p /var/www/.composer
    && chown -R www-data:www-data /var/www/.composer

# PHP-FPM配置优化
RUN { \
    echo "access.log = /dev/null"; \
    echo 'pm.max_children = 100'; \
    echo 'pm.max_requests = 500'; \
} | tee -a /usr/local/etc/php-fpm.d/zz-docker.conf

# 验证安装
RUN ffmpeg -version | head -n1
