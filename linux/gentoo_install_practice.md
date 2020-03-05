# Gentoo Linux 安裝紀錄（練習機）
2020/3/1

接續上篇裝Arch，這篇來裝Gentoo，最後再來決定要裝哪個distro到我的筆電上。因此本篇主要是將Gentoo Linux在練習機上裝到有圖形界面的紀錄。

架構：UEFI+GPT

<table>
  <tr>
    <th>CPU</th>
    <td>Intel(R) Core(TM) i5-4210U CPU @ 1.70GHz</td>
  </tr>
  <tr>
    <th>Cores</th>
    <td>2 cores with hyper-threading</td>
  </tr>
  <tr>
    <th>RAM</th>
    <td>4G</td>
  </tr>
  <tr>
    <th>Disk</th>
    <td>ST1000LM024 HN-M101MBB (HDD)</td>
  </tr>
</table>


## 安裝

主要照著[Gentoo Wiki](https://wiki.gentoo.org/wiki/Handbook:AMD64)的教學一步一步做。


載回`install-amd64-minimal-20200226T214502Z.iso`之後，假設已經成功用隨身碟開機，接下來要連網路。基本上和[Arch載完之後連網路](/linux/arch_install.md#連網路)的方式一樣。

```sh
ip link    # 查看interface，我要用wlp2s0連wifi
ip link set wlp2s0 up
iw dev wlp2s0 scan | grep 'SSID'    #掃一下找要連的wifi
wpa_passphrase wifi名稱 "wifi密碼" > /etc/wpa_supplicant.conf
wpa_supplicant -B -i wlp2s0 -c /etc/wpa_supplicant.conf
dhcpcd wlp2s0
```

測一下有沒有連到網路：
```sh
ping www.gentoo.org
```

校正時間
```sh
ntpd -q -g
```

假設磁碟已經切好也格式化好了，直接來mount吧
```sh
swapon /dev/sda2
mount /dev/sda12 /mnt/gentoo      #new root
```

### stage tarball
```sh
cd /mnt/gentoo
# Download
wget https://bouncer.gentoo.org/fetch/root/all/releases/amd64/autobuilds/20200226T214502Z/stage3-amd64-20200226T214502Z.tar.xz
# Unpack
tar xpvf stage3-*.tar.bz2 --xattrs-include='*.*' --numeric-owner
```

### Portage 設定
範例檔在
`/mnt/gentoo/usr/share/portage/config/make.conf.example`
我們要編輯的檔案是
`/mnt/gentoo/etc/portage/make.conf`
詳細說明參考[Gentoo Wiki](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Stage#Configuring_the_compile_options)，我改了（加了）這些
```sh
COMMON_FLAGS="-march=native -O2 -pipe"
MAKEOPTS="-j5" # CPU cores(4) + 1 = 5
```

### mirror設定
```sh
mirrorselect -i -o >> /mnt/gentoo/etc/portage/make.conf
```
然後用空白鍵選mirror，之後
```sh
mkdir --parents /mnt/gentoo/etc/portage/repos.conf
cp /mnt/gentoo/usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf
```

### Copy DNS info
```sh
cp --dereference /etc/resolv.conf /mnt/gentoo/etc/
```

### Mount the necessary filesystems
```sh
mount --types proc /proc /mnt/gentoo/proc
mount --rbind /sys /mnt/gentoo/sys
mount --make-rslave /mnt/gentoo/sys
mount --rbind /dev /mnt/gentoo/dev
mount --make-rslave /mnt/gentoo/dev
```

### change root到新系統並且mount boot partition
```sh
chroot /mnt/gentoo /bin/bash
source /etc/profile
export PS1="(chroot) ${PS1}"   # 提醒自己是在chroot環境中
mount /dev/sda1 /boot    # EFI partition
```

### Portage 設定
```sh
emerge-webrsync
eselect news list
eselect news read
eselect news purge
eselect profile list
eselect profile set 數字   # 可以選別的，不過我用預設（default/linux/amd64/17.1），此步驟跳過
emerge --ask --verbose --update --deep --newuse @world  ＃ 這步比較久
```
### 設時區
```sh
ls /usr/share/zoneinfo
echo "Asia/Taipei" > /etc/timezone
emerge --config sys-libs/timezone-data
```

### 設locale

在`/etc/locale.gen`裡面寫入
```
en_US.UTF-8 UTF-8
zh_TW.UTF-8 UTF-8
```
之後
```sh
locale-gen
eselect locale list
eselect locale set 數字    # 選en_US.utf8那個
```
之後reload環境
```sh
env-update && source /etc/profile && export PS1="(chroot) ${PS1}"
```

### Kernel
這裡我先偷懶，先用`genkernel`就好，我只是想先在練習機上裝來用用看而已，手動挑核心功能有點過累...之後再弄
```sh
emerge --ask sys-kernel/gentoo-sources
emerge --ask sys-kernel/genkernel
```
改`/etc/fstab`，加入下面這行
```sh
/dev/sda1  /boot  fat32  defaults  0 2  # EFI partition
```
之後
```sh
genkernel all   # 這步超久
emerge --ask sys-kernel/linux-firmware
```

### 改/etc/fstab
```sh
/dev/sda1   /boot        fat32   defaults,noatime     0 2
/dev/sda2   none         swap    sw                   0 0
/dev/sda12  /            ext4    noatime              0 1
/dev/cdrom  /mnt/cdrom   auto    noauto,user          0 0
```

### 調網路設定
改`/etc/conf.d/hostname`裡面的hostname。

裝網路套件
```sh
emerge --ask --noreplace net-misc/netifrc
emerge --ask net-misc/dhcpcd
emerge --ask net-wireless/iw net-wireless/wpa_supplicant
```

### 載System logger
```sh
emerge --ask app-admin/sysklogd
rc-update add sysklogd default
```

### 載Grub
因為我的練習機上已經有其他的Linux載過grub了，所以其實是要先重新開機到載了grub的那個Linux上，記得那個Linux要有`os-prober`，然後跑
```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
之後重新開機就可以開Gentoo了。(感動，Gentoo實在有夠麻煩...)

## 重新開機後

### 刪除stage tarball
```sh
rm /stage3-*.tar.*
```
### 連網路
現在先手動連網，之後等確定要裝在非練習機上再弄network manager之類的。
```sh
ip link    # 查看interface，我要用wlp2s0連wifi
ip link set wlp2s0 up
iw dev wlp2s0 scan | grep 'SSID'    #掃一下找要連的wifi
wpa_passphrase wifi名稱 "wifi密碼" > /etc/wpa_supplicant.conf
wpa_supplicant -B -i wlp2s0 -c /etc/wpa_supplicant.conf
dhcpcd wlp2s0
```
一樣測一下有沒有連到網路
```sh
ping www.gentoo.org
```

### 載sudo、加一般使用者
```sh
emerge --ask app-admin/sudo
useradd -m -G users,wheel,audio -s /bin/bash 使用者名稱
passwd 使用者名稱
visudo
```

### 載圖形界面
```sh
# 圖形界面也是超久
emerge --ask x11-base/xorg-server x11-terms/xterm x11-wm/bspwm x11-misc/sxhkd
```

### bspwm 設定
#### 注意：以下從root身份切換成一般帳號（可以用sudo）
獲取設定檔
```sh
mkdir ~/.config
cd ~/.config
mkdir bspwm
mkdir sxhkd
wget https://raw.githubusercontent.com/baskerville/bspwm/master/examples/bspwmrc
wget https://raw.githubusercontent.com/baskerville/bspwm/master/examples/sxhkdrc
chmod a+x bspwmrc
mv bspwmrc bspwm/
mv sxhkdrc sxhkd/
```
把`~/.config/sxhkd/sxhkdrc`裡面的開啟的terminal emulator設為`xterm`

創`~/.xinitrc`，寫進
```sh
exec bspwm
```
之後`startx`，完成！按super + return就可以開啟xterm
