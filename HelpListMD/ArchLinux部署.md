# Arch Linux 部署开发

## Overview

本文档主要用于记录ayane个人学习arch linux过程，包括如何部署和遇到的问题如何解决

## 1. 安装arch linux

### 1.1 使用 Vmware 创建虚拟机

- 访问(https://wiki.archlinux.org/title/Main_page) arch Linux官网主页，打开安装指南，以此为标准。
- 本文使用 (https://mirrors.bfsu.edu.cn/archlinux/iso/2026.02.01/) 北京外国语大学开源软件镜像站提供的archLinux/iso下载站点。
- 使用下载到的archLinux.iso文件创建虚拟机，虚拟机创建向导询问的模板使用其他Linux 6.x的内核，因为VMware没有为archLinux单独列出来。
- 硬盘存储我分配了60GB。

### 1.2 连接 wifi

1. 罗列出当前系统中所有的“网络接口”以及状态
```bash
ip link
```
2. 连接wifi
- 有线网络直接连接网线
- wifi 使用iwctl
- 移动网卡使用 mmcli 
3. 测试网络连接状态
```bash
ping ping.archlinux.org
```

### 1.3 更新系统时间

确保软件包签名校验成功，防止TLS证书错误
```bash
timedatectl
```
显示结果是UTC + 0 ,和当前的时间不一致，是由于没有设置时区，中国应该是 UTC + 8，可以后续设置回来。

### 1.4 创建硬盘分区

**首先查看当前磁盘设备：**
```bash
fdisk -l
```
或者
```bash
lsblk -l
```
应该会出现两个磁盘。sda 60G 未挂载状态，sr0 1.4G 是系统的.iso镜像文件。

接下来开始分区

**首先创建 boot 分区**
```bash
fdisk /dev/sda
#依次输入
o #新建DOS（MBR）分区表 create a new DOS(MBR)
n #新建分区
p #主分区 primary
1 #编号1
\n #直接回车，从默认起始扇区开始
+512M 
```

**创建根分区 `/`**
```bash
#依次输入
n
p
2
\n #直接回车
\n #直接回车
```

上述分区完成后退出 fdisk
```bash
w
```

**格式化分区：**
本项目创建 Ext4 文件系统
```bash
# 格式化 /boot 分区
mkfs.ext4 /dev/sda1
# 格式化 / 分区
mkfs.ext4 /dev/sda2
```
如果需要创建 FAT32 文件系统 ， 将 `mkfs.ext4` 替换为 `mkfs.fat -F 32` 

**挂载分区**
```bash
# 挂载 / 分区
mount /dev/sda2 /mnt
# 首先创建 /mnt/boot 文件夹路径
mkdir /mnt/boot
# 挂载 /boot  分区
mount /dev/sda1 /mnt/boot
```
*1.我们现在处在创建系统的阶段，所以根目录是 `/mnt` ，实际系统的根目录是 `/`*

*2.`/mnt/boot` 目录是开机启动目录 ， 里面存放 Linux 内核，initramfs-linux.img(启动初始环境) ,启动加载器文件（GRUB）*

## 2. 开始安装系统

---

### 2.1 配置安装软件所用的镜像站列表

方法1：打开archlinux官方提供的镜像站列表生成器 (https://archlinux.org/mirrorlist/) ，选择国家，协议，IP版本之后即可生成

方法2：用指令写入
```bash
curl -L 'https://archlinux.org/mirrorlist/?country=CN&protocol=https' -o /etc/pacman.d/mirrorlist
```
*这里的 `curl` 指令是从网络上抓取内容*

*`-L` 参数是 follow redirect 跟随重定向*

*`-o` 参数是 `output` ，将从 url 抓取到的内容输出到后面的文件*

*url的参数中只有`country=CN&protocol=https` ,没有IP版本的选项，使用默认选项，也就是 ipv4 + ipv6 ,如果需要显式修改，参数部分并上 `&ip_version=4`或`&ip_version=6`*

启用镜像列表（去除 `#` 注释符号）
```bash
sed -i 's/^#Server/Server/' /etc/pacman.d/mirrorlist
```



方法3：先安装 pacman-mirrorlist 获取原始镜像列表再将新的镜像站列表替换掉旧的。也就是说，需要先用旧的镜像站列表安装 pacman-mirrorlist ，可能会很耗时，此处不建议使用。
```bash
pacman -Sy pacman-mirrorlist
```

### 2.2 安装必要的软件包

官方推荐的3个必装的包 `base`, `linux`, `linux-firmware`
```bash
pacstrap -K /mnt base linux linux-firmware
```
*`pacman`和`pacstrap`的区别在于，前者安装到现在正在使用的系统，后者把软件安装到新的系统*

安装过程中报错 `file not found: '/etc/vconsole.conf'`

意思是缺失虚拟终端的配置文件，无伤大雅，但是求稳的话可以手动创建一个。
```bash
# 创建并写入文件内容
cat > /mnt/etc/vconsole.conf << 'EOF'
KEYMAP=us
EOF
# 进入正在创建的 arch linux
arch-chroot /mnt
# 重新构建
mkinitcpio -P
# 退出这个未完成的 linux 系统
exit
```
*`cat` 指令全称是 `concatenate` ，意思是**拼接内容**然后**输出文件内容到屏幕***

*`echo` 是把给他的内容**原样输出***

`|`*管道的作用是把前一个命令的输出作为后一个命令的输入*

`grep` *是过滤器，可以输入文本或者文件，然后输出包含关键词的内容，`-i` 参数表示忽略大小写，`-v`参数表示翻转排除指定内容*

## 3. Configuration System

### 3.1 generate fstab file

*`fstab` 全称是file systems table 文件系统对照表*

```bash
genfstab -U /mnt > /mnt/etc/fstab
```
*这里的 `-U` 是使用 UUID 来标识分区*

*第一个路径参数 `/mnt` 存放未来系统的根目录，从而使用当前路径的 `/mnt/boot`作为未来系统的`/boot`目录。*

*第二个文件路径 `/mnt/etc/fstab` 用来存放目标输出路径。*

*单个大于号 `>`是**覆盖写入**，双大于号 `>>` 是追加写入。*

### 3.2 change root to newly installed system

```bash
arch-chroot /mnt
```

### 3.3 Set time and time zone

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
*`ln -sf` 的 `ln` 是创建 `link`，`-s` 参数是创建**符号连接**，`-f`参数是**强制**的意思，后面两个路径，前者是被指向的路径时区库，后者是链接入口，当前时区规则入口*
*使用 `ln` 创建链接，而非 `cp` 直接复制，优势在于 `ln` 的两个目标可以同步更新*

```bash
hwclock --systohc
```
这条命令用来确保硬件时间被设置到UTC `system to harcware clock`，还有反向操作参数`--hctosys` 将硬件时间设置到系统时间。

### 3.4 region and localization settings

```bash
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

### 3.5 Network configuration

```
echo "ayane" > /etc/hostname
pacman -S networkmanager
```

### 3.6 about initramfs

这个是 linux 系统启动时使用的临时小系统，处于 linux 内核和真正的根文件系统之间。一般不需要重新生成，除非是 LVM ， system encryption ,RAID 。
需要执行：
```bash
mkinitcpio -P
```
*这里的 `-P` 含义是 `preset` ，相当于是**全部***

### 3.7 Set root password

```bash
# 这里设置的是 root 用户的密码
passwd
"input your password"
"input your password again"
```

### install bootloader

```bash
pacman -S grub
grub-install /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
```

## 4. Restart

```bash
reboot
# 刚刚设置的密码是 root 用户密码
root
<设置的密码>
```

不出意外的话，就可以正常进入新系统了

---

**Created by Ayane**

**Last edited : 2026/2/10**
