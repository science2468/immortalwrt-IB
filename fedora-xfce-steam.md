#安装rpm源
```
sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm -y
```
#更新,重启
```
sudo dnf upgrade --refresh
sudo reboot
```
#安装nvidia驱动
```
sudo dnf install akmod-nvidia xorg-x11-drv-nvidia-cuda -y
```
#nvidia驱动签名
```
sudo mokutil --import /etc/pki/akmods/certs/public_key.der
```
#检测驱动
```
modinfo -F version nvidia
```
#ibus输入法：如果 ibus.conf 不存在，手动创建
```
sudo tee /etc/X11/xinit/xinput.d/ibus.conf << 'EOF'
XIM=ibus
XIM_PROGRAM=/usr/bin/ibus-daemon
XIM_ARGS=
GTK_IM_MODULE=ibus
QT_IM_MODULE=ibus
XMODIFIERS=@im=ibus
EOF
```
#安装steam、telegram
```
sudo dnf install steam telegram-desktop -y
```
