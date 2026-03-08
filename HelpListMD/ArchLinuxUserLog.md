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

### 2.3 添加终端中的**提示词联想**和部分代码**高亮**

```bash
sudo pacman -S zsh 
chsh -s /bin/zsh
sudo pacman -S zsh-autosuggestions zsh-syntax-highlighting
```

编辑 `~/.zshrc` 文件，使启动终端时加载上面两个辅助插件

```text
autoload -Uz compinit
compinit

source /usr/share/zsh/plugins/zsh-autosuggestions/zsh-autosuggestions.zsh
source /usr/share/zsh/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
```

重新登录账户，打开终端，启动成功。

### 2.4 系统显示中文异常

```zsh
sudo nano /etc/locale.gen
# 去掉 zh_CN.UTF-8 UTF-8 前面的 #
sudo locale-gen
sudo nano /etc/locale.conf
# 写入一行 `LC_TIME=zh_CN.UTF-8`
```

### 2.5 two-line prompt

Show the Username and workplace in two lines.
The command input-line is the third line.

```bash
vim ~/.zshrc

```
edit the file ,add the follow commands
```text
PROMPT='%F{cyan}%~%f
%F{yellow}%n@%m%f
%# '
```
- %F{color} is to change the color and mark the beginning place
- %f is to end the color
- %n is the username
- %m is the hostname
- %~ is the workplace
- %# show `#` when the user is root and show `%` when the user is common
- 
then resource
```bash
source ~/.zshrc
```

### 2.6 Nvidia GPU Driver install

First of all , check the status of GPU driver

```bash
lspci | grep -E "VGA|3D"
# the VGA represent Intel GPU
# the 3D represent Nvidia GPU
```

Second ,check out which one is in use

```bash
glxinfo | grep "OpenGL"
```

Third , install `nvidia-utils`
```bash
sudo pacman -S nvidia-utils
nvidia-smi
# sometimes it outputs some warnings like "can not communicate with Nvidia Driver"
```

### 2.7 wifi configure GUI

cancel the "#" in the file `/etc/locale.gen` then execute `sudo locale.gen`

```bash
sudo pacman -S waybar network-manager-applet nm-connection-editor
nm-applet
```
then add `nm-applet` to the `hyprland.conf`

### 2.8 Chinese Input

```bash
sudo pacman -S fcitx5 fcitx5-gtk fcitx5-qt fcitx5-configtool fcitx5-rime
# start fcitx5
fcitx5 
# add Input Method rime
fcitx5-configtool
# use `Ctrl + Space` to switch Input Language
# use `Ctrl + `` to switch complex Chinese into simple Chinese
```

