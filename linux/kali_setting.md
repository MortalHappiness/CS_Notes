# Kali Linux Setting

## Sound

https://unix.stackexchange.com/questions/257322/no-audio-on-kali-linux

```
lspci
sudo apt-get install libasound2 alsa-utils alsa-oss
alsamixer
```

### 中文輸入

```
sudo apt install hime
```

### Fonts

```
sudo apt install fonts-symbola fonts-croscore
```

### X11

```
sudo apt install xterm xinput unclutter
```

## Applications

### Basic tools

```
sudo apt install python3-pip
```

### Google Chrome

Go to https://www.google.com/chrome/ and download the `.deb` file, and then

```
sudo apt update
sudo apt install ./google-chrome-stable_current_amd64.deb
```

### Ghidra

Go to https://ghidra-sre.org/ and download the `.zip` file, and then

```
unzip ghidra_9.1.2_PUBLIC_20200212.zip
cd ghidra_9.1.2_PUBLIC_20200212
./ghidraRun
```

### Gdb-peda

https://github.com/longld/peda

```
sudo apt install gdb
git clone https://github.com/longld/peda.git ~/peda
echo "source ~/peda/peda.py" >> ~/.gdbinit
echo "DONE! debug your program with gdb and enjoy"
```

### pwntools

https://github.com/Gallopsled/pwntools

```
pip install pwntools
```

### SageMath

http://ftp.riken.jp/sagemath/linux/64bit/index.html

```
wget http://ftp.riken.jp/sagemath/linux/64bit/sage-9.1-Debian_GNU_Linux_10-x86_64.tar.bz2
tar jxvf sage-9.1-Debian_GNU_Linux_10-x86_64.tar.bz2
cd SageMath
./sage
echo "%colors Linux" > $HOME/.sage/init.sage
# Install pwntools in sagemath
sage -sh
pip install pwntools
```

### pycryptodome

https://github.com/Legrandin/pycryptodome

```
pip install pycryptodome
```

### HashPump

https://github.com/bwall/HashPump

```
git clone https://github.com/bwall/HashPump.git
cd HashPump
make
pip install hashpumpy
```
