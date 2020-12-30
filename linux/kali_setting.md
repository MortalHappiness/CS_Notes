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

### Virtual Box

```
sudo apt install virtualbox
```

### Docker

```
sudo apt install docker.io docker-compose
sudo usermod -aG docker $USER
newgrp - docker
```

### strace/lstrace

```
sudo apt install strace lstrace
```

### Ghidra

Go to https://ghidra-sre.org/ and download the `.zip` file, and then

```
unzip ghidra_9.1.2_PUBLIC_20200212.zip
cd ghidra_9.1.2_PUBLIC_20200212
./ghidraRun
```

### Cutter

https://cutter.re/

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

```sh
wget http://ftp.riken.jp/sagemath/linux/64bit/sage-9.1-Debian_GNU_Linux_10-x86_64.tar.bz2
tar jxvf sage-9.1-Debian_GNU_Linux_10-x86_64.tar.bz2
cd SageMath
./sage
echo "%colors Linux" > $HOME/.sage/init.sage

# Install other python tools in sagemath
sage -sh
pip install pwntools
pip install pycryptodome
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

### gmpy2

https://stackoverflow.com/questions/40075271/gmpy2-not-installing-mpir-h-not-found

```
sudo apt install libgmp-dev libmpfr-dev libmpc-dev
pip install gmpy2
```

### Postman

https://www.postman.com/downloads/

```
wget https://dl.pstmn.io/download/latest/linux64 -O Postman.tar.gz
tar zxvf Postman.tar.gz
cd Postman
./Postman
```

### solc

https://github.com/ethereum/solidity/releases

```
wget https://github.com/ethereum/solidity/releases/download/v0.7.4/solc-static-linux -O solc
```

### Ncat

```
sudo apt install ncat
```

### Ngrok

https://ngrok.com/download

```
wget https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-linux-amd64.zip
unzip ngrok-stable-linux-amd64.zip
```

### scrabble

https://github.com/denny0223/scrabble

```
wget https://raw.githubusercontent.com/denny0223/scrabble/master/scrabble
chmod +x scrabble
```

### mingw-w64

```
sudo apt install mingw-w64
```

### one_gadget / seccomp-tools

https://github.com/david942j/one_gadget
https://github.com/david942j/seccomp-tools

```
gem install --user-install one_gadget
gem install --user-install seccomp-tools
```
```
export PATH="$PATH:$HOME/.gem/ruby/2.7.0/bin"
```
