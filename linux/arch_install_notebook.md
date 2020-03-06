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

編輯`/etc/pacman.d/mirrorlist`，把Taiwan移到最上面。

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

### Windows 雙系統
```sh
sudo pacman -S os-prober
mount /dev/sda1 /mnt
grub-mkconfig -o /boot/grub/grub.cfg
```

### 載一些工具
```sh
sudo pacman -S wget git base-devel
```

### 載圖形界面
```sh
sudo pacman -S xorg xorg-xinit xterm bspwm sxhkd
```

### 設定

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
sudo pacman -S wqy-zenhei    # 中文字體
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
