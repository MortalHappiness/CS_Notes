# Arch Linux 安裝+設定紀錄（筆電）
2020/3/4

比較了Arch和Gentoo後我覺得我還是先裝Arch好了，Gentoo的Portage雖然功能很強，但實在是compile太久了，現在開學了非常需要筆電，萬一哪一步弄錯了要等超久，快速配置好所需的環境應該比較符合我的需求。

設定的部份比較個人，主要是配置成我常用的環境，紀錄下來以便以後重灌可以更快設定好我的環境，可以依自己的需求增減。

---

+ [規劃](#pre-install)
+ [安裝](#install)
+ [設定](#post-install)

---

<span id="pre-install"></span>

## 規劃

UEFI+GPT


<p style="color:red">注意：裝Arch的時候/usr 和/ 必須在同一個partition，否則開機時會出現以下Error：</p>

<pre style="color:red">
Root device mounted successfully, but /sbin/init does not exist.
</pre>

<table style="text-align:center">
  <tr>
    <th colspan="4">/dev/sda (1TB HDD, GPT)</th>
  </tr>
  <tr>
    <th>Name</th>
    <th>Size</th>
    <th>Mount Point</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>sda1</td>
    <td>260M</td>
    <td></td>
    <td>ESP for Windows 10</td>
  </tr>
  <tr>
    <td>sda2</td>
    <td>97.7G</td>
    <td></td>
    <td>Windows 10 (C:)</td>
  </tr>
  <tr>
    <td>sda3</td>
    <td>323G</td>
    <td></td>
    <td>Windows 10 (D:)</td>
  </tr>
  <tr>
    <td>sda4</td>
    <td>8G</td>
    <td>[SWAP]</td>
    <td></td>
  </tr>
  <tr>
    <td>sda5</td>
    <td>30G</td>
    <td>/var</td>
    <td></td>
  </tr>
</table>

<table style="text-align:center">
  <tr>
    <th colspan="4">/dev/sdb (128GB SDD, GPT)</th>
  </tr>
  <tr>
    <th>Name</th>
    <th>Size</th>
    <th>Mount Point</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>sdb1</td>
    <td>260M</td>
    <td>/efi</td>
    <td>ESP</td>
  </tr>
  <tr>
    <td>sdb2</td>
    <td>30G</td>
    <td>/</td>
    <td></td>
  </tr>
  <tr>
    <td>sdb3</td>
    <td>89G</td>
    <td>/home</td>
    <td></td>
  </tr>
</table>

---

<span id="install"></span>

## 安裝

連網路：
```sh
wifi-menu
```

測有沒有連到網路：
```sh
ping archlinux.org
```

校正時間：
```sh
timedatectl set-ntp true
```

mount partitions：
```sh
swapon /dev/sda4
mount /dev/sdb2 /mnt
mkdir /mnt/var
mkdir /mnt/efi
mkdir /mnt/home
mount /dev/sda5 /mnt/var
mount /dev/sdb1 /mnt/efi
mount /dev/sdb3 /mnt/home
```

編輯`/etc/pacman.d/mirrorlist`，參考[官方教學](https://wiki.archlinux.org/index.php/Mirrors)。
我是先到[Mirrorlist Generator](https://www.archlinux.org/mirrorlist/)勾http,https,IPv4後複製一份到`/etc/pacman.d/mirrorlist.backup`，之後把Worldwide和Taiwan全部取消註解。之後`pacman -S pacman-contrib`，再`rankmirrors  /etc/pacman.d/mirrorlist.backup > /etc/pacman.d/mirrorlist`

載kernel和firmware：
```sh
pacstrap /mnt base linux linux-firmware
```

生成fstab，生成完檢查一下對不對：
```sh
genfstab -U /mnt >> /mnt/etc/fstab
```

change root到新系統：
```sh
arch-chroot /mnt
```

載vim：
```sh
pacman -S vim
```

### 時區
```sh
ln -sf /usr/share/zoneinfo/Asia/Taipei /etc/localtime
hwclock --systohc
```

### locale
把`/etc/locale.gen`裡面的`en_US.UTF-8 UTF-8`、`zh_TW.UTF-8 UTF-8`、`zh_TW.BIG5`的註解拿掉之後：
```sh
locale-gen
```
創`/etc/locale.conf`，在裡面加`LANG=en_US.UTF-8`。

### 網路設定
創`/etc/hostname`並寫入自己的hostname。
創`/etc/hosts`並寫入：
```
127.0.0.1	localhost
::1		localhost
127.0.1.1	myhostname.localdomain	myhostname
```
其中兩個`myhostname`為剛才寫在`/etc/hostname`裡的名字。

裝網路套件：
```sh
pacman -S dhcpcd iw wpa_supplicant
```

### 載Grub
```sh
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id='Arch Linux'
```

修改`/etc/default/grub`，我自己在`GRUB_CMDLINE_LINUX_DEFAULT`(Kernel parameter的設定)加上`sysrq_always_enabled=1`，以便可以用`REISUB`重新開機。

#### Hibernate 設定
1. Kernel parameter須加上`resume=UUID=...`，指向SWAP。https://wiki.archlinux.org/index.php/Power_management/Suspend_and_hibernate#Required_kernel_parameters
2. `/etc/mkinitcpio.conf`的`HOOKS`那行要加進`resume`。https://wiki.archlinux.org/index.php/Power_management/Suspend_and_hibernate#Configre_the_initramfs
3. `mkinitcpio -p linux`
https://wiki.archlinux.org/index.php/Mkinitcpio#Image_creation_and_activation

```sh
# Windows 雙系統
pacman -S os-prober
mount /dev/sda1 /mnt
sudo timedatectl set-local-rtc 1
# 生成grub.cfg
grub-mkconfig -o /boot/grub/grub.cfg
```

### 改root密碼
```sh
passwd
```

### 創一般使用者，給予sudo權限
```sh
pacman -S sudo
useradd -m -s /bin/bash 使用者名稱
passwd 使用者名稱
export EDITOR=vim
visudo
```

之後重新開機，用一般使用者登入。

### 連網路
```sh
ip link    # 查看interface，我要用wlp3s0連wifi
sudo ip link set wlp3s0 up
sudo iw dev wlp2s0 scan | grep 'SSID'    #掃一下找要連的wifi
sudo wpa_passphrase wifi名稱 "wifi密碼" > /etc/wpa_supplicant.conf
sudo wpa_supplicant -B -i wlp3s0 -c /etc/wpa_supplicant.conf
sudo dhcpcd wlp3s0
```

測有沒有連到網路：
```sh
ping archlinux.org
```

---

<span id="post-install"></span>

## 設定

### 載一些工具
```sh
sudo pacman -S wget git base-devel zip unzip ctags
```

### 載圖形界面和相關工具
```sh
sudo pacman -S xorg xorg-xinit xterm termite bspwm sxhkd brightnessctl dzen2 scrot sxiv alsa-utils unclutter
```

### gitub ssh 設定
```sh
sudo pacman -S openssh
```
之後參考[教學](https://help.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh)

### 關於複製貼上的一些工具
```sh
pacman -S xclip clipnotify clipmenu
```
註：xclip的使用方法為
```sh
xclip -sel clip < 檔案名稱
```
它可以把檔案讀進`CLIPBOARD`。

### 讓`video`群組的人可以調亮度
用root權限編輯
```
ACTION=="add", SUBSYSTEM=="backlight", KERNEL=="名稱", RUN+="/bin/chgrp video /sys/class/backlight/%k/brightness"
ACTION=="add", SUBSYSTEM=="backlight", KERNEL=="名稱", RUN+="/bin/chmod g+w /sys/class/backlight/%k/brightness"
```
其中"名稱"要看`ls /sys/class/backlight`的輸出是什麼，例如`acpi_vedio0`或`intel_backlight`。這個設定要重新開機才有效 。

如果要讓使用者可以調螢幕亮度，請把使用者加進`vedio`群組。
```sh
sudo usermod -a -G vedio 使用者名稱
```

### 圖形介面設定

clone [我自己的Linux設定檔](https://github.com/MortalHappiness/Linux-config)到`$HOME`：

```sh
cd ~/
git clone https://github.com/MortalHappiness/Linux-config.git
```

cd 進去並執行install的shell script，詳細請見`README.md`
```sh
cd Linux-config
bash setup.sh install
```

之後`startx`，按super + return就可以開啟xterm。

### 載字體
```sh
sudo pacman -S ttf-liberation   # free metric-compatible substitute for fonts in Windows and Microsoft Office
sudo pacman -S ttf-croscore     # Chrome OS 字體
sudo pacman -S wqy-zenhei       # 中文字體
sudo pacman -S ttf-font-awesome # icon
sudo pacman -S noto-fonts-emoji # emoji
```

### 中文輸入法
```sh
sudo pacman -S gcin
```

### 載Google-Chrome

```sh
git clone https://aur.archlinux.org/google-chrome.git
cd google-chrome
makepkg -si
# 其他distribution都直接用google-chrome當指令，習慣了，故創一個soft link
ln -s /usr/bin/google-chrome-stable /usr/bin/google-chrome
google-chrome
```

### 載Miniconda
```sh
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
```

### 載dart-sass

```sh
# 載到$HOME
wget https://github.com/sass/dart-sass/releases/download/1.26.2/dart-sass-1.26.2-linux-x64.tar.gz
tar zxvf dart-sass-1.26.2-linux-x64.tar.gz
rm zxvf dart-sass-1.26.2-linux-x64.tar.gz
```
編輯.bashrc

```sh
export PATH="$PATH:$HOME/dart-sass"
```

### 載nodejs
```sh
sudo pacman -S nodejs npm
```

#### 或載nvm
```sh
git clone https://aur.archlinux.org/nvm.git
cd nvm
makepkg -si
```

#### fzf
超好用的fuzzy finder
```sh
sudo pacman -S fzf
```
然後`.bashrc`裡加兩行
```sh
source /usr/share/fzf/key-bindings.bash
source /usr/share/fzf/completion.bash
```

### PDF Reader
[Foxit Reader](https://www.foxitsoftware.com/pdf-reader/)
載下來之後解壓縮跑install的script之後再加到PATH就行了。

### Postman
https://www.postman.com/downloads/

