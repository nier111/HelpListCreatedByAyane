
2. **获取 Windows 主机在 WSL2 中的 IP**：
```bash
# 在 WSL2 中执行
ip route show | grep default | awk '{print $3}'
# 输出类似：172.17.32.1
```

3. **修改 OpenClaw 配置**，添加代理：
```bash
# 编辑配置文件
nano ~/.openclaw/openclaw.json
```

在 `channels.telegram` 部分添加：
```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "botToken": "你的BotToken",
      "proxy": "http://172.17.32.1:7890",  // ← 添加这行
      "dmPolicy": "allowlist",
      "allowFrom": [你的UserID]
    }
  }
}
```

⚠️ **注意**：
- `172.17.32.1` 换成你实际获取到的 IP
- `7890` 换成你的代理端口（Clash 默认 7890，V2Ray 可能是 1080）
- **不要在全局设置 HTTP_PROXY**，只在 channel 配置里设置！

---

## 六、常见问题解决

### 问题 1：Telegram 能发不能收（收不到消息）

**现象**：能收到 bot 回复的配对码，但发消息没反应

**原因**：OpenClaw 默认使用 webhook 接收消息，但 WSL2 没有公网 IP

**解决**：使用 Polling 模式（长轮询）

1. 删除现有的 webhook：
```bash
curl "https://api.telegram.org/bot<你的BotToken>/deleteWebhook?drop_pending_updates=true"
```

2. 修改配置为白名单模式（跳过配对）：
```json
# 配置文件路径 ： ~/.openclaw/openclaw.json
{
  "channels": {
    "telegram": {
      "dmPolicy": "allowlist",      // ← 改为 allowlist
      "allowFrom": [user id]     // ← 填入你的 User ID
    }
  }
}
```

3. 重启 OpenClaw 服务

### 问题 2：内网穿透端口不对

**现象**：使用樱花/花生壳等内网穿透，设置 webhook 时报错

**原因**：Telegram webhook 只接受 **80, 88, 443, 8443** 端口

**解决**：要么改隧道端口为 443/8443，要么用 Polling 模式（推荐）

### 问题 3：Gateway Token 不匹配

**现象**：页面能打开但无法聊天，日志显示 token 错误

**原因**：URL 中的 token 和配置不匹配

**解决**：执行 `pnpm openclaw dashboard --no-open` 后，复制链接中的 token 到配置中。

---

## 七、常用命令

| 命令 | 作用 |
|------|------|
| `pnpm openclaw onboard` | 重新运行配置向导 |
| `pnpm openclaw status` | 查看当前状态 |
| `pnpm openclaw logs --follow` | 查看实时日志 |
| `pnpm openclaw pairing list` | 查看待批准的配对请求 |
| `pnpm openclaw pairing approve <CODE>` | 批准配对码 |
| `Ctrl + C` | 中断当前进程 |
| `pnpm openclaw dashboard --no-open` | 返回包含token的访问控制台链接 |
| `pnpm openclaw gateway` | 开启gateway服务（启动openclaw） |
---

## 八、安全建议

1. **保持私人模式**：
   - `dmPolicy: "allowlist"` 而不是 `"open"`
   - 只添加信任的 User ID 到 `allowFrom`

2. **不要泄露**：
   - Bot Token
   - API Key
   - Gateway Token（URL 中的 tokens 参数）

3. **群组使用**：
   - 默认 `groupPolicy: "allowlist"` 更安全
   - 添加群组 ID 到白名单后再使用

---

## 九、配置文件示例

完整配置参考（`~/.openclaw/openclaw.json`）：

```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "botToken": "123456789:ABCDEF...",
      "proxy": "http://172.17.32.1:7890",
      "dmPolicy": "allowlist",
      "allowFrom": [your telegram id],
      "groupPolicy": "allowlist",
      "streamMode": "partial"
    }
  },
  "gateway": {
    "port": 18789,
    "mode": "local",
    "bind": "loopback"
  }
}
```

---

**最后更新**：2026-02-07
**作者**：Ayane + 八爪鱼 🐙
