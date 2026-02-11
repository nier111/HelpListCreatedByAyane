# 树莓派手动wifi配置

## Overview
本项目是为了实现，在树莓派没有连接外部wifi的情况下能够主动开启wifi热点服务，此时我们用手机连接这个热点，访问本地服务，可以帮助树树莓派进行wifi配置，以便进行SSH远程登录。

本项目实验过程使用的是树莓派4B版本

## 环境/依赖安装

```bash
sudo apt update
sudo apt upgrade
sudo apt install -y python3 python3-pip python3-flask network-manager dnsmasq
```

## 安装步骤

### 1.文件编写

- 创建项目安装路径
```bash
mkdir -p ~/projects/autoWifiConnect/templates
cd ~/projects/autoWifiConnect
```

- 编写 wifi.py 代码
 ```bash
cat > ~/projects/autoWifiConnect/wifi.py << 'EOF'
import subprocess
import time
import logging

IFACE = "wlan0"
AP_NAME = "Pi-AP"
TARGET_CON = "WiFi-Target"

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(message)s"
)

def sh(cmd):
    logging.info("RUN: %s", " ".join(cmd))
    r = subprocess.run(cmd, capture_output=True, text=True)
    if r.stdout:
        logging.info("STDOUT: %s", r.stdout.strip())
    if r.stderr:
        logging.info("STDERR: %s", r.stderr.strip())
    logging.info("RC=%d", r.returncode)
    return r

def ap_up():
    sh(["nmcli", "connection", "up", AP_NAME])

def ap_down():
    sh(["nmcli", "connection", "down", AP_NAME])

def _parse_wifi_list_tsv(text):
    out = []
    for line in text.splitlines():
        if not line.strip():
            continue

        # 先统一把 \: 变回 :
        line = line.replace("\\:", ":").replace("\\", "")

        cols = line.split(":")
        # BSSID 6段 + SSID + SIGNAL + SECURITY = 至少 9 段
        if len(cols) < 9:
            continue

        bssid = ":".join(cols[0:6]).lower()
        ssid = cols[6]
        signal = cols[7]
        security = cols[8]

        out.append({"bssid": bssid, "ssid": ssid, "signal": signal, "security": security})
    return out

def scan_wifi(max_items=20):
    # 扫描阶段：临时关 AP
    ap_down()
    sh(["nmcli", "device", "set", IFACE, "managed", "yes"])
    sh(["nmcli", "radio", "wifi", "on"])
    sh(["nmcli", "dev", "wifi", "rescan"])
    time.sleep(2)

    r = sh(["nmcli", "-t", "-f", "BSSID,SSID,SIGNAL,SECURITY", "dev", "wifi", "list"])
    aps = _parse_wifi_list_tsv(r.stdout)

    # 同 SSID 取信号最强
    best = {}
    for ap in aps:
        ssid = ap["ssid"]
        if not ssid:
            continue
        if ssid not in best:
            best[ssid] = ap
        else:
            try:
                if int(ap["signal"]) > int(best[ssid]["signal"]):
                    best[ssid] = ap
            except:
                pass

    result = sorted(best.values(), key=lambda x: int(x["signal"]) if x["signal"].isdigit() els>

    # 扫描结束恢复 AP
    ap_up()
    return result[:max_items]

def _get_ap_info_by_bssid(bssid):
    """
    用 nmcli 列表里查 BSSID 对应的 SSID/SECURITY
    """
    r = sh(["nmcli", "-t", "-f", "BSSID,SSID,SECURITY", "dev", "wifi", "list"])
    aps = _parse_wifi_list_tsv(r.stdout.replace("BSSID:SSID:SECURITY", ""))
    # 上面复用解析器时缺 signal 字段会不够列，因此这里改用更稳的方式：
    # 直接再跑带 signal 的命令
    r2 = sh(["nmcli", "-t", "-f", "BSSID,SSID,SIGNAL,SECURITY", "dev", "wifi", "list"])
    aps2 = _parse_wifi_list_tsv(r2.stdout)
    for ap in aps2:
        if ap["bssid"].lower() == bssid.lower():
            return ap
    return None

def _guess_key_mgmt(security):
    """
    根据 SECURITY 字段选择 key-mgmt
    常见：
      WPA2 / WPA1 WPA2 -> wpa-psk
      WPA3 / SAE       -> sae
      空 / --          -> none（开放网络）
    """
    sec = (security or "").upper()
    if "SAE" in sec or "WPA3" in sec:
        return "sae"
    if "WPA" in sec:
        return "wpa-psk"
    return "none"

def connect_wifi_by_bssid(bssid, password, timeout=25):
    logging.info("CONNECT request: bssid=%s", bssid)

    # 连接前：关 AP
    ap_down()
    sh(["nmcli", "radio", "wifi", "on"])

    # 重新扫描一次，确保能查到该 BSSID（AP 模式下列表可能变化）
    sh(["nmcli", "dev", "wifi", "rescan"])
    time.sleep(2)

    apinfo = _get_ap_info_by_bssid(bssid)
    if not apinfo:
        ap_up()
        return False, f"未找到该热点（bssid={bssid}）"

    ssid = apinfo["ssid"]
    security = apinfo["security"]
    keymgmt = _guess_key_mgmt(security)

    logging.info("AP resolved: ssid=%s security=%s keymgmt=%s", ssid, security, keymgmt)

    # 删除旧的目标连接（避免残留）
    sh(["nmcli", "con", "delete", TARGET_CON])

    # 新建连接：锁定 BSSID，避免同名/乱码问题
    sh(["nmcli", "con", "add",
        "type", "wifi",
        "ifname", IFACE,
        "con-name", TARGET_CON,
        "ssid", ssid
    ])
    sh(["nmcli", "con", "modify", TARGET_CON, "802-11-wireless.bssid", bssid])

    if keymgmt == "none":
        sh(["nmcli", "con", "modify", TARGET_CON, "ipv4.method", "auto", "ipv6.method", "ignor>
    else:
        sh(["nmcli", "con", "modify", TARGET_CON, "802-11-wireless-security.key-mgmt", keymgmt>
        sh(["nmcli", "con", "modify", TARGET_CON, "802-11-wireless-security.psk", password])
        sh(["nmcli", "con", "modify", TARGET_CON, "802-11-wireless-security.psk-flags", "0"])
        sh(["nmcli", "con", "modify", TARGET_CON, "ipv4.method", "auto", "ipv6.method", "ignor>

    # 启动连接
    r = sh(["nmcli", "con", "up", TARGET_CON])
    if r.returncode != 0:
        ap_up()
        return False, "连接启动失败（可能密码错误/加密不匹配）"

    # 等待 wlan0 connected
    for _ in range(timeout):
        time.sleep(1)
        s = sh(["nmcli", "-t", "-f", "DEVICE,STATE,CONNECTION", "dev"]).stdout
        if f"{IFACE}:connected:{TARGET_CON}" in s:
            return True, "connected"

    # 超时回退
    ap_up()
    return False, "连接超时，已恢复热点"

EOF
 ```

- 编写 web.py 文件，网页前端
```bash
cat > ~/projects/autoWifiConnect/web.py << 'EOF'
from flask import Flask, render_template, request
import wifi
import logging
import re

logging.basicConfig(level=logging.INFO)

app = Flask(__name__)

def clean_bssid(x: str) -> str:
    x = (x or "").strip()
    x = x.replace("\\:", ":")     # 把 \: 还原成 :
    x = x.replace("\\", "")       # 去掉其它反斜杠
    x = x.replace("：", ":")      # 全角冒号
    x = re.sub(r"[^0-9a-fA-F:]", "", x)  # 去掉不可见字符
    return x.lower()

@app.route("/", methods=["GET", "POST"])
def index():
    if request.method == "POST":
        app.logger.info("FORM=%s", dict(request.form))

        pwd = request.form.get("password", "")
        bssid = clean_bssid(request.form.get("bssid", ""))

        if not re.fullmatch(r"([0-9a-f]{2}:){5}[0-9a-f]{2}", bssid):
            return f"非法BSSID: {repr(bssid)}", 400

        ok, detail = wifi.connect_wifi_by_bssid(bssid, pwd)  # 用你的wifi.py函数名
        return ("连接成功，热点将关闭，请切回家里Wi-Fi用SSH连接树莓派"
                if ok else f"连接失败：{detail}")

    wifi_list = wifi.scan_wifi()
    return render_template("index.html", wifi_list=wifi_list)

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=80, debug=False)

EOF
```

- 编写网页页面 templates/index.html
```bash
cat > ~/projects/autoWifiConnect/templates/index.html << 'EOF'
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <title>Pi Wi-Fi 配置</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <style>
    body{font-family:system-ui,Arial; padding:16px; max-width:520px; margin:auto;}
    select,input,button{width:100%; padding:10px; margin:8px 0; font-size:16px;}
    .msg{white-space:pre-wrap; background:#f3f3f3; padding:10px; border-radius:8px;}
    .small{opacity:.7; font-size:13px;}
  </style>
</head>
<body>
  <h2>Wi-Fi 配置</h2>

  {% if msg %}
    <div class="msg">{{ msg }}</div>
  {% endif %}

  <form method="post">
    <label>选择 Wi-Fi（信号强度%）</label>
    <select name="bssid" required>
      {% for w in wifi_list %}
        <option value="{{ w.bssid }}">{{ w.ssid }} ({{ w.signal }}%)</option>
      {% endfor %}
    </select>

    <label>密码</label>
    <input name="password" type="password" placeholder="无密码请留空">

    <button type="submit">连接</button>
    <div class="small">提示：括号里的百分比是信号强度，不是成功率。</div>
  </form>
</body>
</html>

EOF
```
### 2. 用 nmcli 创建热点连接 Pi + AP (开机用)

```bash
#清理同名残留
nmcli -t -f NAME connection show | grep -x "Pi-AP" >/dev/null && sudo nmcli connection delete "Pi-AP" || true
#创建AP
sudo nmcli connection add type wifi ifname wlan0 con-name Pi-AP autoconnect yes ssid Pi-Setup
sudo nmcli connection modify Pi-AP 802-11-wireless.mode ap 802-11-wireless.band bg
sudo nmcli connection modify Pi-AP ipv4.method shared ipv6.method ignore
sudo nmcli connection modify Pi-AP wifi-sec.key-mgmt wpa-psk wifi-sec.psk "raspberry123"
#启动AP
sudo nmcli connection up Pi-AP
```
### 3. 创建 systemd 服务：开机自启 web 配网页面

```bash
cat > /tmp/autowifi.service << 'EOF'
[Unit]
Description=Auto WiFi Config Portal
After=NetworkManager.service network-online.target
Wants=network-online.target

[Service]
Type=simple
User=root
WorkingDirectory=/home/ayane/projects/autoWifi
ExecStart=/usr/bin/python3 /home/ayane/projects/autoWifi/web.py
Restart=always
RestartSec=2
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
EOF

sudo cp /tmp/autowifi.service /etc/systemd/system/autowifi.service
sudo systemctl daemon-reload
sudo systemctl enable autowifi
sudo systemctl restart autowifi
```

## 验证

### 1. 服务是否常驻

运行
```bash
systemctl status autowifi
```
应该看到
`Active: active (running)`

### 2. 端口是否常驻
运行
```bash
sudo ss -lntp | grep ':80'
```
应该看到`0.0.0.0：80`被python3监听

### 3. AP 是否正常

运行
 ```bash
nmcli dev status
nmcli con show --active
 ```
手机连接WiFi：`Pi-Setup`，密码是`raspberry123`
在此之前，要先在树莓派上用获取到它的 ip 地址
```bash
ifconfig
```
再使用浏览器打开`wlan0`的ip地址：`10.42.0.1`

## 其他调试用指令

### 1. 服务启动失败后的“失败限速”解除

```bash
sudo systemctl reset-failed autowifi
```

### 2. 查看详细报错日志

```bash
sudo journalctl -u autowifi -f
```

### 3. 中断当前已连接的wifi，模拟断网操作

```bash
#列出当前连接的wifi
sudo nmcli dev status
#断开连接
sudo nmcli device disconnect wlan0
```

### 4. 查看设备连接状态

```bash
 nmcli con show
```

## 手机能监测到wifi信号，但是连接显示Pi-Setup拒绝请求

### 1. 强制清空 wlan0 状态

```bash
sudo nmcli device disconnect wlan0 || true
sudo nmcli radio wifi off
sudo sleep 2
sudo nmcli radio wifi on
```

### 2. 设置国家码

```bash
sudo raspi-config
```
路径：
```choice
-> localisation Options
-> WLAN Country
-> CN 
```

## 调试总结

主要问题都出在数据格式匹配上
- 中文名称的ssid有时候会被解析成乱码
- bssid 被接收到树莓派的时候，`:` 会被转义成 `\:`，会出现多余的 `\`，需要进行数据清理 


**Last edit : 2026/2/8**
**Created by ayane**
