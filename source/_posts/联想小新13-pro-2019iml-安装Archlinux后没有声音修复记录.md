---
title: 联想小新13 pro 2019iml 安装Archlinux后没有声音修复记录
date: 2022-06-22 11:51:42
tags: Arch Linux
---

## 安装pulseaudio、pulseaudio-alsa后系统显示无声音

在联想小新13 pro 2019iml安装win10&&Archlinux双系统后，archlinux下没有声音

确认声卡

```bash
sudo lspci | grep audio
sudo lspci -v
```

经测试，系统没有读取到声卡

## 解决

修改grub文件内容

```bash
export EDITOR=vim
sudoedit /etc/grub/grub
```

在如下位置进行修改

```bash
GRUB_CMDLINE_LINUX_DEFAULT="${原有配置不要动} snd_hda_intel.dmic_detect=0"
```

保存后退出重启即可

或者安装sof-firmware包：

```bash
sudo pacman -S sof-firmware
```
