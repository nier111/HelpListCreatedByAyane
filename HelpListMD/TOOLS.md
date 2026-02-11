# TOOLS.md - Local Notes

## Telegram Bot (WSL2)
- 使用 Windows 主机 IP 作为代理地址，不是 `host.docker.internal`
- Clash 必须开启 "Allow LAN" 才能被 WSL2 访问
- 代理格式: `http://<windows-ip>:7890`

## Previous Issues
- 第一代配置了全局 HTTP_PROXY 环境变量，导致 gateway 启动异常
- 正确做法：只在 channel 配置里设置 proxy，不改全局环境变量
