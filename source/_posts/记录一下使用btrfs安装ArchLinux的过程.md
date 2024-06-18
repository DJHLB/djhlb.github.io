---
title: 记录一下使用btrfs安装ArchLinux的过程
date: 2024-05-25 19:26:54
tags: ArchLinux
---

#### 1. 事前准备

下载并验证安装文件：[archlinux-RELEASE_VERSION-x86_64.iso](https://archlinux.org/download/)

插上合适的安装介质(U盘等)，使用dd命令来制作：

```bash
sudo dd if=archlinux-RELEASE_VERSION-x86_64.iso of=/dev/sdx bs=4M status=progress oflag=sync
```

完成之后，就可以从安装介质启动了

#### 2. 开始安装

##### 2.1 reflector

首先禁用reflector：

```bash
systemctl stop reflector.service
```

##### 2.2 键盘布局和字体

列出可用的键盘布局：

```bash
localectl list-keymaps
```

使用你想要的键盘布局：

```bash
loadkeys your-keymap
```

默认键盘布局是`us` ，我们使用默认即可

切换终端字体，例如 *terminus*`ter-v24n`:

```bash
setfont ter-v24n
```

##### 2.3 连接网络

有线一般无需操作；无线的话，使用iwctl命令进行连接：

```bash
iwctl                           #执行iwctl命令，进入交互式命令行
device list                     #列出设备名，比如无线网卡看到叫 wlan0
station wlan0 scan              #扫描网络
station wlan0 get-networks      #列出网络 比如想连接YOUR-WIRELESS-NAME这个无线
station wlan0 connect YOUR-WIRELESS-NAME #进行连接 输入密码即可
exit                            #成功后exit退出
```

测试一下有没有网络：

```bash
ping www.gnu.org
```

##### 2.4 更新系统时钟

```bash
timedatectl set-ntp true    #将系统时间与网络时间进行同步
timedatectl status          #检查服务状态
```

##### 2.5 磁盘分区

列出设备上的存储设备：`lsblk -f`，然后设置一个变量来标记要安装在的硬盘（我的是nvme0n1）：

```bash
export disk="/dev/nvme0n1"
```

删除旧的分区：

```bash
wipefs -af $disk
sgdisk --zap-all --clear $disk
partprobe $disk
```

可选：用随机数据填充硬盘

临时加密：

```bash
cryptsetup open --type plain -d /dev/urandom $disk target
```

填充：

```bash
dd if=/dev/zero of=dev/mapper/target bs=1M status=progress oflag=direct
```

移除：

```bash
cryptsetup close target
```

现在来开始分区，这里我使用的是`sgdisk` 命令：

```bash
sgdisk --lsit-types		#列出相关系统文件类型的代码
sgdisk -n 0:0:+800MiB -t 0:ef00 -c 0:esp $disk		#创建EFI分区
sgdisk -n 0:0:0 -t 0:8309 -c 0:luks $disk		#创建根目录
partprobe $disk
```

打印新的分区表：

```bash
sgdisk -p $disk
```

###### 2.5.1 磁盘加密

我这里使用LUKS1

###### 2.5.2 初始化加密分区：

```bash
cryptsetup --type luks1 -v -y luksFormat ${disk}p2		#这里p2是根目录
```

###### 2.5.3 格式化分区

```bash
cryptsetup open ${disk}p2 cryptdev
mkfs.vfat -F32 -n ESP ${disk}p1
mkfs.btrfs -L archlinux /dev/mapper/cryptdev
```

###### 2.5.4 挂载根目录并创建btrfs子卷

```bash
mount /dev/mapper/cryptdev /mnt
```

创建子卷：

```bash
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@snapshots
btrfs subvolume create /mnt/@cache
btrfs subvolume create /mnt/@root
btrfs subvolume create /mnt/@srv
btrfs subvolume create /mnt/@log
btrfs subvolume create /mnt/@tmp
```

##### 2.6 挂载分区

首先先卸载根目录分区：

```bash
umount /mnt
```

设置一下子卷挂载参数选项，方便后续操作：

```bash
export sv_opts="rw,noatime,compress-force=zstd:1,space_cache=v2"
```

然后先挂载btrfs根子卷：

```bash
mount -o ${sv_opts},subvol=@ /dev/mapper/cryptdev /mnt
```

为其他子卷创建挂载点：

```bash
mkdir -p /mnt/{home,root,.snapshots,srv,var/cache,var/log,var/tmp}
```

挂载其他子卷：

```bash
mount -o ${sv_opts},subvol=@home /dev/mapper/cryptdev /mnt/home
mount -o ${sv_opts},subvol=@root /dev/mapper/cryptdev /mnt/root
mount -o ${sv_opts},subvol=@snapshots /dev/mapper/cryptdev /mnt/.snapshots
mount -o ${sv_opts},subvol=@cache /dev/mapper/cryptdev /mnt/var/cache
mount -o ${sv_opts},subvol=@srv /dev/mapper/cryptdev /mnt/srv
mount -o ${sv_opts},subvol=@log /dev/mapper/cryptdev /mnt/var/log
mount -o ${sv_opts},subvol=@tmp /dev/mapper/cryptdev /mnt/var/tmp
```

最后别忘了挂载ESP分区：

```bash 
mkdir /mnt/efi
mount ${disk}p1 /mnt/efi
```

##### 2.7 安装系统

一些必要的包：

```bash
pacstrap /mnt base base-devel linux linux-headers linux-firmware
```

其他功能性包：

```bash
pacstrap /mnt bash-completion cryptsetup man-db vim wget curl git networkmanager pacman-contrib pkgfile sudo
```

##### 2.8 Fstab

```bash
genfstab -U -p /mnt >> /mnt/etc/fstab
```

##### 3 基础配置

首先进入系统：

```bash
arch-chroot /mnt /bin/bash
```

##### 3.1 时区

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc
```

##### 3.2 主机名

编辑`/etc/hostname`设置主机名

```bash
vim /etc/hostname
```

编辑`/etc/hosts`并加入以下内容：

```bash
127.0.0.1   localhost
::1         localhost
127.0.1.1   your_hostname
```

##### 3.3 设置Locale

```bash
export locale="en_US.UTF-8"
export localezh="zh_CN.UTF-8"
sed -i "s/^#\(${locale}\)/\1/" /etc/lcoale.gen
sed -i "s/^#\(${localezh}\)/\1/" /etc/locale.gen
echo "LANG=${locale}" > /etc/local.conf
locale-gen
```

##### 3.4 为root用户设置密码

```bash
passwd root
```

##### 3.5 安装微码

```bash
pacman -S intel-ucode   #Intel
pacman -S amd-ucode     #AMD
```

##### 3.6 创建新用户

```bash
useradd -m -G wheel -s /bin/bash foo
passwd foo
```

为用户foo设置root权限：

```bash
sed -i "s/# %wheel ALL=(ALL:ALL) ALL/%wheel ALL=(ALL:ALL) ALL/" /etc/sudoers
```

##### 3.7 NetworkManager

设置`NetworkManager`开机启动：

```bash
systemctl enable NetworkManager
```

##### 3.8 配置Keyfile

```bash
dd bs=512 count=4 iflag=fullblock if=/dev/random of=/crypto_keyfile.bin
chmod 600 /crypto_keyfile.bin
```

将keyfile添加至LUKS：

```bash
cryptsetup luksAddKey ${disk}p2 /crypto_keyfile.bin
```

##### 3.9 Mkinitpio

编辑`/etc/mkinitcpio.conf`文件：

1. 添加keyfile：

   ```bash
   FILES=(/crypt_keyfile.bin)
   ```

2. 添加`btrfs`支持：

   ```bash
   MODULES=(btrfs)
   ```

3. 设置hooks：

   ```bash
   HOOKS=(base udev keyboard autodetect keymap consolefont modconf block encrypt filesystems fsck)
   ```

最后需要`mkinitcpio -P`一下

##### 3.10 安装引导程序

```bash
pacman -S grub efibootmgr
```

因为使用了luks加密，这里需要配置一下grub

首先找到加密硬盘的UUID：

```bash
blkid -s UUID -o value ${disk}p2
```

然后编辑`/etc/default/grub`，定位到`GRUB_CMDLINE_LINUX_DEFAULT`这一行并填写：

```bash
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet cryptdevice=UUID=UUID_OF_ENCRYPTED_PARTITION:cryptdev"
```

这里将`UUID=UUID_OF_ENCRYPTED_PARTITION`替换为你加密硬盘的UUID

添加`luks`模块：

```bash
GRUB_PRELOAD_MODULES="part_gpt part_msdos luks"
```

为grub启用`GRUB_ENABLE_CRYPTODISK`：

```bash
GRUB_ENABLE_CRYPTODISK=y
```

安装：

```bash
grub-install --target=x86_64-efi --efi-directory=/efi --boot-directory=/efi --bootloader-id=GRUB
```

最后生成 GRUB 所需的配置文件：

```bash
grub-mkconfig -o /efi/grub/grub.cfg
```

##### 4 完成安装

```bash
exit
umount -R /mnt
reboot
```

##### 5 其他设置

###### 5.1 字体

fonts.conf:

```
  <!-- Default font for the zh_CN locale (no fc-match pattern) -->
  <match>
    <test compare="contains" name="lang">
      <string>zh_CN</string>
    </test>
    <edit mode="prepend" name="family">
      <!-- 更改下一行即可, 以下同理 -->
      <string>PingFang SC</string>
    </edit>
  </match>

  <!-- 默认无衬线字体 -->
  <!-- Default sans-serif font -->
  <match target="pattern">
    <test qual="any" name="family">
      <string>sans-serif</string></test>
    <edit name="family" mode="prepend" binding="same">
      <string>Inter</string>
    </edit>
  </match>

  <!-- 默认衬线字体 -->
  <!-- Default serif fonts -->
  <match target="pattern">
    <test qual="any" name="family">
      <string>serif</string>
    </test>
    <edit name="family" mode="prepend" binding="same">
      <string>STSong</string>
    </edit>
  </match>

  <!-- 默认等宽字体 -->
  <!-- Default monospace fonts -->
  <match target="pattern">
    <test qual="any" name="family">
      <string>monospace</string>
    </test>
    <edit name="family" mode="prepend" binding="same">
      <string>MesloLGS Nerd Font Mono</string>
    </edit>
  </match>

<!-- Fallback fonts preference order -->
  <alias>
    <family>sans-serif</family>
    <prefer>
      <family>Inter</family>
      <family>PingFang SC</family>
      <family>PingFang TC</family>
      <family>Microsoft YaHei UI</family>
      <family>Microsoft YaHei</family>
      <family>Noto Sans CJK SC</family>
      <family>Noto Sans CJK TC</family>
      <family>Noto Sans CJK JP</family>
      <family>Noto Sans CJK KR</family>
      <family>Noto Color Emoji</family>
      <family>Noto Emoji</family>
    </prefer>
  </alias>
  <alias>
    <family>serif</family>
    <prefer>
      <family>STSong</family>
      <family>SimSun</family>
      <family>Noto Serif</family>
      <family>Noto Serif CJK SC</family>
      <family>Noto Serif CJK TC</family>
      <family>Noto Serif CJK JP</family>
      <family>Noto Serif CJK KR</family>
      <family>Noto Color Emoji</family>
      <family>Noto Emoji</family>
    </prefer>
  </alias>
  <alias>
    <family>monospace</family>
    <prefer>
      <family>MesloLGS Nerd Font Mono</family>
      <family>Microsoft YaHei UI</family>
      <family>Microsoft YaHei</family>
      <family>Noto Color Emoji</family>
      <family>Noto Emoji</family>
    </prefer>
  </alias>
```

###### 5.2 snapshots 和 system rollbacks

首先安装snapper:

```bash
sudo pacman -S snapper snap-pac
```

在前面安装时已经创建了`@snapshots`子卷，先卸载并删除其挂载点：

```bash
sudo umount /.snapshots
sudo rm -rf /.snapshots
```

然后让snapper生成默认的配置：

```bash
sudo snapper -c root create-config /
```

删除由snapper创建的挂载点，自己重新创建一个并挂载它：

```bash
sudo btrfs subvolume delete .snapshots
sudo mkdir /.snapshots
sudo mount -a
```

这样设置就把 Snapper 创建的所有快照存储在 @ 子卷之外了。然后可以替换 @ 而不会丢失快照。

设置一下权限（注意用户须有root权限来使用快照）：

```bash
sudo chmod 750 /.snapshots
sudo chown :wheel /.snapshots
```

接下来设置定时快照，编辑snapper配置文件`/etc/snapper/configs/root`：

```bash
[......]
ALLOW_USERS="foo"	#允许用户foo管理快照
[......]
TIMELINE_MIN_AGE="1800"
TIMELINE_LIMIT_HOURLY="5"
TIMELINE_LIMIT_DAILY="7"
TIMELINE_LIMIT_WEEKLY="0"
TIMELINE_LIMIT_MONTHLY="0"
TIMELINE_LIMIT_YEARLY="0"
```

设置snapper自启动：

```bash
sudo systemctl enable --now snapper-timeline.timer
sudo systemctl enable --now snapper-cleanup.timer
```

配置Updatedb（系统有locate命令），编辑`/etc/updatedb.conf`添加一行：`PRUNENAMES = ".snapshots"`

为Grub添加快照选项，首先安装`grub-btrfs`包：

```bash
sudo pacman -S grub-btrfs
```

在 `/etc/default/grub-btrfs/config` 中设置包含 `grub.cfg` 文件的目录位置：

```bash
GRUB_BTRFS_GRUB_DIRNAME="/efi/grub"
```

再设置一下Grub自动更新：

```bash
sudo systemctl enable --now grub-btrfs.path
```

接下来设置只读快照，从快照启动系统时将是只读模式。在 `/etc/mkinitcpio.conf` 中 HOOKS 末尾添加 `grub-btrfs-overlayfs`：

```bash
HOOKS=(base ... fsck grub-btrfs-overlayfs)
```

重新生成initramfs：

```bash
sudo mkinitcpio -P
```

手动回滚：  启动到由overlayfs 提供的快照挂载读写后，挂载顶级子卷(subvolid=5)。  也就是说，省略任何 subvolid 或 subvol 安装标志（例如：标记为 cryptdev 的加密设备映射）：

```bash
sudo mount /dev/mapper/cryptdev /mnt
```

移动并标记损坏的@子卷：

```bash
sudo mv /mnt/@ /mnt/@.broken
```

或者干脆删除@子卷：

```bash
sudo btrfs  /mnt/@
```

找到要恢复的快照的编号：

```bash
sudo grep -r '<date>' /mnt/@snapshots/*/info.xml
[...]
/.snapshots/8/info.xml:  <date>2022-08-20 15:21:53</date>
/.snapshots/9/info.xml:  <date>2022-08-20 15:22:39</date> 
```

创建只读快照：

```bash
sudo btrfs subvolume snapshot /mnt/@snapshots/number/snapshot /mnt/@	#number为要恢复的快照的编号
```

卸载/mnt。  

重启并回滚！



###### 参考：

[A(rch) to Z(ram): Install Arch Linux with (almost) full disk encryption and BTRFS](https://www.dwarmstrong.org/archlinux-install/)

[[Arch Linux 安装使用教程 - ArchTutorial - Arch Linux Studio](https://archlinuxstudio.github.io/ArchLinuxTutorial/#/?id=arch-linux-安装使用教程-archtutorial-arch-linux-studio)](https://archlinuxstudio.github.io/ArchLinuxTutorial/#/)
