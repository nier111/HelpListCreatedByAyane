# esp-idf 环境配置部署

## 1. Overview

本项目旨在在 vscode 中安装 esp-idf 插件并配置环境，用于开发 esp32 相关项目

## 2. 环境依赖安装

### 2.1 Vscode 端

安装 ESP-IDF 和 Clangd 插件

### 2.2 终端

安装必要插件：
```bash
sudo pacman -S git wget flex bison gperf python python-pip cmake ninja dfu-util picocom base-devel python-virt-idualenv

mkdir ~/tools/esp
cd ~/tools/esp
git clone https://github.com/espressif/esp-idf.git
cd esp-idf
./install.sh
source ./export.sh
# if there is no mistakes and warinings,the installation is OK.
```