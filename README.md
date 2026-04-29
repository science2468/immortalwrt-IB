# 使用openwrt的记录
### 1：给[固件选择器](https://firmware-selector.openwrt.org)添加了roofs_size_mb的参数，可以在构建前填入≤1024MB大小的数字，生成填写数字大小的固件。可以下载后在本地使用。
`www`的文件夹就是前端。构建成功后的文件下载速度很慢 10kb/s 左右，原因就是pcdn。
下载到本地使用之前先看下[openwrt的源代码](https://github.com/openwrt/firmware-selector-openwrt-org)或者[immortalwrt的源代码](https://github.com/immortalwrt/firmware-selector-immortalwrt-org)，下载后将`www`文件夹内的文件替换下就可以用了
### tunsafe在openwrt25.10-re-sp-01b上编译成功
### usque masque-port： 443 500 1701 4500 4443 8443 8095


```
git clone -b openwrt-25.12 https://github.com/openwrt/openwrt.git
```
cd openwrt/ && ./scripts/getver.sh
```
./scripts/feeds update -a && ./scripts/feeds install -a
```
git switch -c v25.12.2-xg-040g-md
```
git fetch origin pull/21896/head:pr-21896
```
git checkout pr-21896
```
git show db150718c3 --stat
```
make target/linux/prepare V=s > /tmp/make_prepare.log 2>&1
```
