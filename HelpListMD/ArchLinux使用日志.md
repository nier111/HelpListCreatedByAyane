# Arch Linux 使用日志

---

## Overview

---

本文件主要用于记录使用 arch Linux 过程中遇到的 Bug 和经验总结
 
---

## Bugs

---

### `1.1 ping www.baidu.com` 报错 `temporay failure in name resolution`

- 1. NetworkManager 没在工作
```bash
# 查看 NetworkManager 当前工作状态，是不是 dead
systemctl status NetworkManager
# 启动 NetworkManager 服务
systemctl start NetworkManager
# 设置 NetworkManager 为开机自启动
systemctl enable NetworkManager
```

---

## 使用体验优化

---

### 2.1 光标切换不丝滑

- 1. 调整 tty 键盘重复率(不行)
```bash
kbdrate -d 200 -r 30
```
- 2. 安装图形桌面
```bash
#  测试网络连通性
ping archlinux.org
# 安装图形基础
pacman -S xorg-server xorg-xinit xorg-apps
# 安装 KDE plasma
pacman -S plasma
# 安装显示器管理器（登录页面）
pacman -S sddm
systemctl enable sddm
# 安装vmware 需要的工具
pacman -S open-vm-tools
systemctl enable vmtoolsd
# 安装终端(图像化页面中使用 Ctrl + Alt + T 来调出终端)
pacman -S konsole
# 重启
reboot
```

### 2.2 vscode中文字体包不起作用:安装中文字体

```bash
sudo pacman -S noto-fonts noto-fonts-cjk noto-fonts-emoji
fc-cache -fv
reboot
```

