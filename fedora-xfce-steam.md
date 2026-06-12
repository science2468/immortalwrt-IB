#更新,重启
```
sudo dnf upgrade --refresh
```
#nvidia驱动签名
```
sudo dnf install kmodtool akmods mokutil openssl
```
```
sudo kmodgenca -a
```
```
sudo mokutil --import /etc/pki/akmods/certs/public_key.der
```
```
systemctl reboot
```
#安装rpm源
```
sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm -y
```

#安装nvidia驱动
```
sudo dnf install akmod-nvidia xorg-x11-drv-nvidia-cuda -y
```

#检测驱动
```
modinfo -F version nvidia
```
```
sudo reboot
```

#安装steam、telegram
```
sudo dnf install steam telegram-desktop -y
```
#ibus输入法：如果 ibus.conf 不存在，手动创建
```
sudo dnf install ibus-panel -y

sudo tee /etc/X11/xinit/xinput.d/ibus.conf << 'EOF'
XIM=ibus
XIM_PROGRAM=/usr/bin/ibus-daemon
XIM_ARGS=
GTK_IM_MODULE=ibus
QT_IM_MODULE=ibus
XMODIFIERS=@im=ibus
EOF

imsettings-boot.sh
```
### 卸载旧内核
```
sudo dnf remove kernel*x.x.x*
```
