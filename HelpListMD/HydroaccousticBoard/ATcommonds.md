# AT Commands

## Overview

本文档主要针对 4G通信模块 EG800AK-CN 的 `AT` 指令进行归纳总结和声明。

## 1. 硬件连接

- 1. 4G模块的，VIN和GND接12V电源。
- 2. 4G模块的TXD、RXD、GND。

## 2. AT 指令列表

| AT 指令 | 预期返回值 | 说明 |
| --- | --- | --- |
| `AT` | OK | 设备复位 |
| `AT+CPIN?` | `+CPIN: READY` , `OK`| 检查SIM卡状态 |
| `AT+CSQ` | `+CSQ: <rssi>,<ber>` , `OK` | 获取信号强度,rssi值越大，ber值越小信号越好 |
| `AT+COPS?` | `+COPS:0,0,"CHINA MOBILE"，7` , `OK` | 检查注册的网络 |
| `AT+CGATT?` | `+CGATT: 1`, `OK` | 检查GPSR附着状态 |
| `AT+CREG?` | `+CREG: 0,1` 或 `+CREG: 0,5` , `OK` | 查询网络注册状态 |
| `AT+CGDCONT=1,"IP","CMNET"`| `OK` | 设置APN |  
| `AT+CGDCONT?` | `+CGDCONT: 1,"IP","CMNET"` , `OK` | 查询APN状态 |
| `AT+QIACT=1` | `OK` | 激活PDP上下文（激活网络连接） |
| `AT+QIACT?` | `+QIACT: 1,1,1,"IP地址"` , `OK` | 检查网络连接状态,显示获取到的IP地址 |
| `AT+QIOPEN=1,0,"TCP","目标IP",端口,0,1` | `+QIOPEN: 0,0` , `OK` | TCP连接测试 |
| `AT+QISEND=0` | `> ` | 等待发送数据,在终端使用 `Ctrl+Z` 结束输入 |
| `AT+QISTATE=1,0` | `+QISTATE: 0,"TCP","117.68.10.96",59969,0,2,1,0,1,"uart1"` , `OK` | 查询TCP连接状态 |
| `AT+QICLOSE=0` | `OK` | 关闭TCP连接 |

## 3.  终端实例测试

使用串口工具打开4G模块的UART串口，并设置波特率为115200，进行交互。

```bash
AT
OK

AT+QIACT=1
ERROR # 有时候报错ERROR是因为设备开机已经激活了PDP，被反复激活导致报错，可以忽略

AT+QIACT?
+QIACT: 1,1,1,"10.89.179.111"


AT+QIOPEN=1,0,"TCP","117.68.10.96",59969,0,1
OK
+QIOPEN: 0,0
# 如果返回 `+QIOPEN: 0,563` ,可能是先前已经激活了一次TCP，没有关闭，被反复激活导致报错，可以忽略

AT+QISEND=0
> hello
`Ctrl+Z`  # 键盘输入结束符
SEND OK

AT+QISTATE=1,0
+QISTATE: 0,"TCP","117.68.10.96",59969,0,2,1,0,1,"uart1"
OK

AT+QICLOSE=0
OK
# 只有当结束当前的TCP连接，python的终端才能输出接收到的数据
```

## 4. 测试程序

### localHost.py 监听端

该程序用于监听主机本地IP的5002端口，接收来自4G模块的TCP数据。如果需要更改端口号，修改 port=5002 即可。

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/api/sensor', methods=['POST'])
def sensor():
    data = request.get_json(force=True, silent=True)
    print("收到数据:", data)
    return jsonify({"status": "ok"}), 200

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5002)

```

### SendTest.py 发送端

该程序用于通过serial串口连接4G模块，并向4G模块的UART串口发送数据。如果需要更改IP地址或端口号，修改 ip='192.168.10.1' 和 port=5002 即可。

使用前需要将程序中的串口号`PORT`、波特率`BAUDRATE`、`APN`、目标IP`SERVER_IP`、目标端口`SERVER_PORT`、发送数据`SEND_DATA`修改为实际值。

```python
import serial
import time

# =========================
# 需要参数
# =========================
PORT = "COM11"
BAUDRATE = 115200

APN = "CMNET"

SERVER_IP = "117.68.10.96"         # 目标IP
SERVER_PORT = 59969            # 目标端口
SEND_DATA = "hello from EG800AK\r\n"   # 你要发送的数据

# 连接号
CONNECT_ID = 0


def send_at(ser, cmd, timeout=3):
    """发送AT命令并打印响应"""
    ser.reset_input_buffer()
    ser.write((cmd + "\r\n").encode())
    ser.flush()

    start = time.time()
    resp = b""

    while time.time() - start < timeout:
        if ser.in_waiting:
            resp += ser.read(ser.in_waiting)
            if b"OK\r\n" in resp or b"ERROR\r\n" in resp:
                break
        time.sleep(0.05)

    text = resp.decode("utf-8", errors="ignore")
    print(f"\n>>> {cmd}")
    print(text.strip() if text.strip() else "<no response>")
    return text


def wait_qiopen_result(ser, conn_id=0, timeout=20):
    """等待QIOPEN结果"""
    start = time.time()
    resp = ""

    while time.time() - start < timeout:
        if ser.in_waiting:
            chunk = ser.read(ser.in_waiting).decode("utf-8", errors="ignore")
            resp += chunk
            print(chunk, end="")

            if f"+QIOPEN: {conn_id},0" in resp:
                return True
            if "ERROR" in resp:
                return False
            if f"+QIOPEN: {conn_id}," in resp and f"+QIOPEN: {conn_id},0" not in resp:
                return False

        time.sleep(0.1)

    return False


def send_data(ser, data, conn_id=0):
    """用AT+QISEND发送数据"""
    ser.reset_input_buffer()
    ser.write((f"AT+QISEND={conn_id}\r\n").encode())
    ser.flush()

    start = time.time()
    prompt = ""

    while time.time() - start < 5:
        if ser.in_waiting:
            prompt += ser.read(ser.in_waiting).decode("utf-8", errors="ignore")
            if ">" in prompt:
                break
            if "ERROR" in prompt:
                print("进入发送模式失败")
                print(prompt)
                return False
        time.sleep(0.05)

    if ">" not in prompt:
        print("没有收到 > 提示符")
        return False

    # 发送实际数据
    ser.write(data.encode("utf-8"))
    time.sleep(0.2)

    # Ctrl+Z 结束发送
    ser.write(b"\x1A")
    ser.flush()

    time.sleep(2)

    result = ""
    if ser.in_waiting:
        result = ser.read(ser.in_waiting).decode("utf-8", errors="ignore")

    print("\n>>> SEND RESULT")
    print(result.strip() if result.strip() else "<no response>")

    if "SEND OK" in result or "OK" in result:
        return True
    return False


def main():
    ser = None
    try:
        ser = serial.Serial(
            port=PORT,
            baudrate=BAUDRATE,
            timeout=1,
            write_timeout=2
        )
        time.sleep(0.2)

        # 1. 基础检查
        if "OK" not in send_at(ser, "AT", 2):
            print("模块无响应")
            return

        if "READY" not in send_at(ser, "AT+CPIN?", 3):
            print("SIM卡异常")
            return

        if ",1" not in send_at(ser, "AT+CREG?", 3):
            print("网络未注册")
            return

        if "+CSQ:" not in send_at(ser, "AT+CSQ", 3):
            print("无法读取信号强度")
            return

        # 2. 配APN
        if "OK" not in send_at(ser, f'AT+QICSGP=1,1,"{APN}","","",1', 5):
            print("APN配置失败")
            return

        # 3. 激活网络
        send_at(ser, "AT+QIDEACT=1", 5)
        time.sleep(1)

        if "OK" not in send_at(ser, "AT+QIACT=1", 10):
            print("网络激活失败")
            send_at(ser, "AT+QIGETERROR", 3)
            return

        send_at(ser, "AT+QIACT?", 3)

        # 4. 建TCP连接
        send_at(ser, f"AT+QICLOSE={CONNECT_ID}", 3)
        time.sleep(1)

        cmd = f'AT+QIOPEN=1,{CONNECT_ID},"TCP","{SERVER_IP}",{SERVER_PORT},0,1'
        ser.reset_input_buffer()
        ser.write((cmd + "\r\n").encode())
        ser.flush()

        print(f"\n>>> {cmd}")

        if not wait_qiopen_result(ser, CONNECT_ID, 20):
            print("TCP连接失败")
            send_at(ser, "AT+QIGETERROR", 3)
            send_at(ser, f"AT+QISTATE=1,{CONNECT_ID}", 3)
            return

        print("TCP连接成功")

        # 5. 发数据
        if send_data(ser, SEND_DATA, CONNECT_ID):
            print("数据发送成功")
        else:
            print("数据发送失败")

        # 可选：查询状态
        send_at(ser, f"AT+QISTATE=1,{CONNECT_ID}", 3)

        # 6. 关闭连接
        send_at(ser, f"AT+QICLOSE={CONNECT_ID}", 3)

    except Exception as e:
        print("运行异常：", e)
    finally:
        if ser and ser.is_open:
            ser.close()


if __name__ == "__main__":
    main()
```

## 5. SakuraFRP 内网穿透工具

由于4G模块需要通过公共互联网进行通信，而不是简单的局域网内通信，所以我们需要使用内网穿透工具进行端口映射，将本地端口映射到公网上。我们可以使用免费的SakuraFRP工具进行内网穿透，获得可以临时使用的公网IP和端口，方法如下：

- 1. 访问官网：(https://www.natfrp.com/)
- 2. 注册账号
- 3. 下载对应操作系统的客户端软件
- 4. 在官网主页，找到用户密钥，输入客户端软件登录
- 5. 在隧道标签下，点击 `+` ，新建隧道
- 6. 选择`TCP`协议，节点，输入`本地端IP`和`目标端口`, 比如我们测试用的端口号是5002
- 7. 点击`创建隧道` 
- 8. 在日志标签下，从输出日志中找到映射到的公网IP和端口，比如`172.16.31.10:10000`



