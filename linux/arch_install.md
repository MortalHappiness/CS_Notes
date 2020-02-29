## Arch Linux 安裝紀錄
2020/2/29

Linux Mint用了一年，雖然安裝非常簡單，開箱即用，Cinnamon也很好看，但它預先裝了太多不會用到的東西了，而且最近想試試輕量的bspwm tiling window manager，所以先跟Mint說掰掰，來用號稱超flexible的Gentoo或Arch。(Mint之後就放隨身碟當救援用的OS吧)

我不知道Arch和Gentoo哪個比較好用，那就都來裝裝看用用看再來決定吧！大家都說Gentoo要make很久，那就先來裝Arch，本篇主要是將Arch Linux在練習機上裝到有圖形界面的紀錄。

架構：UEFI+GPT

## 安裝

主要照著[Arch Wiki](https://wiki.archlinux.org/index.php/Installation_guide)的教學一步一步做。

假設已經成功用隨身碟開機，接下來要連網路。安裝時最簡單連網的方法應該是直接
```sh
wifi-menu
```
連wifi就好了。

測一下有沒有連到網路：
```sh
ping archlinux.org
```

設一下時間
```sh
timedatectl set-ntp true
```

假設磁碟已經切好也格式化好了，直接來mount吧
```sh
swapon /dev/sda2
mount /dev/sda11 /mnt       #new root
mkdir /mnt/efi
mount /dev/sda1 /mnt/efi    #efi partition
```

編輯`/etc/pacman.d/mirrorlist`，把Taiwan移到最上面。

載kernel和firmware
```sh
pacstrap /mnt base linux linux-firmware
```

生成fstab
```sh
genfstab -U /mnt >> /mnt/etc/fstab
```
生成完檢查一下`/mnt/etc/fstab`看對不對。

change root到新系統
```sh
arch-chroot /mnt
```

載vim不然甚至沒文字編輯器...
```sh
pacman -S vim
```

### 調時間
```sh
ln -sf /usr/share/zoneinfo/Asia/Taipei /etc/localtime
hwclock --systohc
```

### 調locale
把`/etc/locale.gen`裡面的`en_US.UTF-8 UTF-8`、`zh_TW.UTF-8 UTF-8`、`zh_TW.BIG5`的註解拿掉之後
```sh
locale-gen
```
創`/etc/locale.conf`，在裡面加`LANG=en_US.UTF-8`。

### 調網路設定（我覺得這個最麻煩）
創`/etc/hostname`並寫入自己的hostname。
創`/etc/hosts`並寫入
```
127.0.0.1	localhost
::1		localhost
127.0.1.1	myhostname.localdomain	myhostname
```
其中兩個`myhostname`為剛才寫在`/etc/hostname`裡的名字。

裝網路套件
```sh
pacman -S dhcpcd iw wpa_supplicant
```

### 載Grub
因為我的練習機上已經有其他的Linux載過grub了，所以其實是要先重新開機到載了grub的那個Linux上，記得那個Linux要有`os-prober`，然後跑
```
grub-mkconfig -o /boot/grub/grub.cfg
```
之後重新開機就可以開Arch了。

## 重新開機後

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
ping archlinux.org
```

### 載圖形界面
```sh
pacman -S xorg xorg-xinit xterm bspwm sxhkd
```

#### bspwm 設定
```sh
pacman -S wget sudo
```

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

## 加碼：載Google-Chrome並正常顯示中文(順便試用AUR)
```sh
sudo pacman -S git base-devel
git clone https://aur.archlinux.org/google-chrome.git
cd google-chrome
makepkg -si
pacman -S wqy-zenhei    # 中文字體
google-chrome-stable
```
