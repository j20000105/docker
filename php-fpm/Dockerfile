FROM php:fpm

# 安装依赖、扩展和工具
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
    # 安装 FFmpeg
    && case "$(dpkg --print-architecture)" in \
    amd64) \
    FFMPEG_URL="https://github.com/BtbN/FFmpeg-Builds/releases/download/latest/ffmpeg-master-latest-linux64-gpl.tar.xz" \
    ;; \
    arm64) \
    FFMPEG_URL="https://github.com/BtbN/FFmpeg-Builds/releases/download/latest/ffmpeg-master-latest-linuxarm64-gpl.tar.xz" \
    ;; \
    *) echo "Unsupported architecture: $(dpkg --print-architecture)"; exit 1 ;; \
    esac \
    && wget -q "${FFMPEG_URL}" -O ffmpeg.tar.xz \
    && tar -xf ffmpeg.tar.xz --strip-components=1 \
    && mv bin/ffmpeg bin/ffprobe /usr/local/bin/ \
    && chmod +x /usr/local/bin/ffmpeg /usr/local/bin/ffprobe \
    && rm -rf ffmpeg* *.tar.xz \
    # 清理
    && apt-get purge -y --auto-remove wget xz-utils \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Composer
COPY --from=composer/composer:latest-bin /composer /usr/bin/composer

# 配置
COPY custom.ini /usr/local/etc/php/conf.d/custom.ini
COPY zz-docker.conf /usr/local/etc/php-fpm.d/zz-docker.conf

# 验证安装
RUN composer --version | head -n1
RUN ffmpeg -version | head -n1
