# Kali Linux Setting

## Sound

https://unix.stackexchange.com/questions/257322/no-audio-on-kali-linux

```
lspci
sudo apt-get install libasound2 alsa-utils alsa-oss
alsamixer
```

## Applications

### Google Chrome

Go to https://www.google.com/chrome/ and download the `.deb` file, and then

```
sudo apt update
sudo apt install ./google-chrome-stable_current_amd64.deb
```
