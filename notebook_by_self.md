使用(RIPEstatAPI)[https://stat.ripe.net/docs/data-api/ripestat-data-api] 查询
powershell：Invoke-RestMethod -Uri "https://stat.ripe.net/data/dns-chain/data.json?resource=www.google.com" | ConvertTo-Json




asu本地部署：
用debian13或者是Ubuntu24的非root用户
1
```bash
# Install uv
curl -LsSf https://astral.sh/uv/install.sh | sh

# Install dependencies
uv sync --extra dev
```
2
```bash
sudo apt install -y podman-compose git

# 克隆asu
git clone --depth 1 https://github.com/openwrt/asu.git /home/debian/
```
3
```bash
# systemd
systemctl --user enable podman.socket
systemctl --user start podman.socket
systemctl --user status podman.socket
```
4
```bash
echo "PUBLIC_PATH=$(pwd)/public" > .env
echo "CONTAINER_SOCKET_PATH=/run/user/$(id -u)/podman/podman.sock" >> .env
# optionally allow custom scripts running on first boot
echo "ALLOW_DEFAULTS=1" >> .env
```
5
```bash
cd asu
# use existing containers
podman-compose pull

# build containers locally
podman-compose build

# start asu
podman-compose up -d

#show containers
podman-compose ps
```

配套的firmware-selector-openwrt-org 仓库最要紧的是default -> nginx的配置文件在 `/etc/nginx/sites-enabled/`





```shell

./scripts/feeds update -a && ./scripts/feeds install -a

cat > .config << EOF
          CONFIG_TARGET_airoha=y
          CONFIG_TARGET_airoha_an7581=y
          CONFIG_TARGET_airoha_an7581_DEVICE_nokia_xg-140g-md=y
          EOF

./scripts/getver.sh

nohup make -j $(nproc) V=s > /tmp/make.log 2>&1 &


#测试patch
make -j $(nproc) target/linux/{clean,prepare} V=s

```