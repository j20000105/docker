# Docker images

该仓库自动跟踪上游最新版，并在验证通过后发布多架构镜像。上游构建失败或
smoke test 失败时，不会更新公开的 `latest` 标签。

## Images

| Image | Platforms | Upstream policy |
| --- | --- | --- |
| `j20000105/php-fpm` | `linux/amd64`, `linux/arm64` | 每日重建浮动的 PHP-FPM、Composer、PECL 扩展和 FFmpeg master |
| `j20000105/logstash` | `linux/amd64`, `linux/arm64` | 每日从 Elastic 官方 registry 发现最高稳定 semver |

## Publishing

- 各架构先推送至固定 staging 标签并执行运行时测试，避免临时标签无限累积。
- 测试通过后按 digest 合并并原子更新 `latest`。
- 两个镜像分别使用原生 amd64 与 arm64 runner，避免在 QEMU 中编译扩展或安装插件。
- FFmpeg 校验文件及 PECL 版本作为缓存键，上游变化会精确触发对应层重建。
- PHP-FPM 使用 FFmpeg GPL shared 构建，在保留编解码器的同时复用公共库以减小镜像。
- PHP-FPM 同时保留 `sha-<git-sha>`、`run-<run-id>-<attempt>` 标签。
- Logstash 同时保留上游版本和 `sha-<git-sha>` 标签。
- GitHub Actions 固定到完整 commit SHA，并由 Dependabot 跟踪更新。

GitHub 仓库需要配置 `DOCKERHUB_USERNAME` variable 和
`DOCKERHUB_TOKEN` secret。

## Runtime notes

- PHP-FPM 默认启用 OPcache 文件时间戳检查，适用于挂载或原地更新应用代码。
- PHP-FPM 使用 dynamic process manager；请按容器内存限制调整 worker 数量。
- Logstash 保留官方镜像的完整 `logstash.yml`。不要把未配置认证与 TLS 的
  9600 端口暴露到不受信任网络。
