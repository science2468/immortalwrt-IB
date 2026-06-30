```
apk-mbedtls base-files ca-bundle dnsmasq-full dropbear firewall4 fstools kmod-crypto-hw-eip93 kmod-gpio-button-hotplug kmod-leds-gpio kmod-nft-offload libc libgcc libustream-mbedtls logd mtd netifd nftables odhcp6c odhcpd-ipv6only ppp ppp-mod-pppoe procd-ujail uboot-envtools uci uclient-fetch urandom-seed urngd wpad-basic-mbedtls kmod-mt7603 kmod-mt7615-firmware kmod-mmc-mtk kmod-usb3 luci luci-i18n-attendedsysupgrade-zh-cn luci-i18n-package-manager-zh-cn luci-i18n-base-zh-cn luci-i18n-firewall-zh-cn luci-i18n-pbr-zh-cn kmod-tun luci-proto-wireguard
```
```
mkdir -p /etc/tunsafe && \
wget -O /usr/bin/tunsafe https://github.com/lmq8267/tunsafe/releases/download/2025-10-09/tunsafe_aarch64-linux && \
cat > /etc/init.d/tunsafe << 'EOF'
#!/bin/sh /etc/rc.common
# Copyright (C) 2025 OpenWrt.org

START=95
STOP=10
USE_PROCD=1

PROG=/usr/bin/tunsafe
CONFIG_DIR=/etc/tunsafe
CONFIG_FILE=$CONFIG_DIR/tunsafe.conf
PID_FILE=/var/run/tunsafe.pid

start_service() {
    procd_open_instance
    procd_set_param command "$PROG" start "$CONFIG_FILE"
    procd_set_param respawn
    procd_set_param stdout 1
    procd_set_param stderr 1
    procd_set_param pidfile "$PID_FILE"
    procd_close_instance
}

stop_service() {
    # procd 会自动停止
    true
}

reload_service() {
    stop
    start
}
EOF
chmod +x /usr/bin/tunsafe /etc/init.d/tunsafe
```
```
# Beware! This script will be in /rom/etc/uci-defaults/ as part of the image.
# Uncomment lines to apply:
#
wlan_name="OpenWrt"
wlan_password="12345678"
#
# root_password=""
#
# For 24.10 and earlier, the 'netmask' will default to '255.255.255.0':
# lan_ip_address="192.168.1.1"
#
# But, make sure to use CIDR notation for 25.12 and later:
# lan_ip_address="192.168.1.1/24"
#
pppoe_username="307"
pppoe_password="307307307"

# log potential errors
exec >/tmp/setup.log 2>&1

if [ -n "$root_password" ]; then
  (echo "$root_password"; sleep 1; echo "$root_password") | passwd > /dev/null
fi

# Configure LAN
# More options: https://openwrt.org/docs/guide-user/base-system/basic-networking
if [ -n "$lan_ip_address" ]; then
  uci -q batch << EOI
  set network.lan.ipaddr="$lan_ip_address"
  commit network
  EOI
fi

# Configure WLAN
# More options: https://openwrt.org/docs/guide-user/network/wifi/basic#wi-fi_interfaces
if [ -n "$wlan_name" -a -n "$wlan_password" -a ${#wlan_password} -ge 8 ]; then
  uci -q batch << EOI
  set wireless.@wifi-device[0].disabled='0'
  set wireless.@wifi-device[0].channel='auto'
  set wireless.@wifi-device[0].country='CN'
  set wireless.@wifi-iface[0].disabled='0'
  set wireless.@wifi-iface[0].encryption='psk2'
  set wireless.@wifi-iface[0].ssid="$wlan_name"
  set wireless.@wifi-iface[0].key="$wlan_password"
  set wireless.@wifi-device[1].disabled='0'
  set wireless.@wifi-device[1].htmode='VHT80'
  set wireless.@wifi-device[1].channel='auto'
  set wireless.@wifi-device[1].country=''CN
  set wireless.@wifi-iface[1].disabled='0'
  set wireless.@wifi-iface[1].encryption='psk2'
  set wireless.@wifi-iface[1].ssid="$wlan_name"
  set wireless.@wifi-iface[1].key="$wlan_password"
  commit wireless
  EOI
fi

# Configure PPPoE and route
# More options: https://openwrt.org/docs/guide-user/network/wan/wan_interface_protocols#protocol_pppoe_ppp_over_ethernet
if [ -n "$pppoe_username" -a "$pppoe_password" ]; then
  uci -q batch << EOI
  set network.wan.proto=pppoe
  set network.wan.username="$pppoe_username"
  set network.wan.password="$pppoe_password"
  add network route
  set network.@route[-1].interface='protonvpn'
  set network.@route[-1].target='8.8.8.8/32'
  commit network
  EOI
fi

# Configure dhcp
cat > /etc/dnsmasq.servers << 'EOF'
server=/google.com/8.8.8.8
server=/google.com.hk/8.8.8.8
server=/googleapis.com/8.8.8.8
server=/googleusercontent.com/8.8.8.8
server=/googlevideo.com/8.8.8.8
server=/youtube.com/8.8.8.8
server=/youtu.be/8.8.8.8
server=/ytimg.com/8.8.8.8
server=/ggpht.com/8.8.8.8
server=/twitter.com/8.8.8.8
server=/t.co/8.8.8.8
server=/twimg.com/8.8.8.8
server=/facebook.com/8.8.8.8
server=/fb.com/8.8.8.8
server=/fbcdn.net/8.8.8.8
server=/instagram.com/8.8.8.8
server=/cdninstagram.com/8.8.8.8
server=/messenger.com/8.8.8.8
server=/telegram.org/8.8.8.8
server=/telegra.ph/8.8.8.8
server=/proton.me/8.8.8.8
server=/chatgpt.com/8.8.8.8
server=/x.com/8.8.8.8
server=/reddit.com/8.8.8.8
server=/redditmedia.com/8.8.8.8
server=/redditstatic.com/8.8.8.8
server=/redd.it/8.8.8.8
server=/steamcommunity.com/8.8.8.8
EOF

uci -q batch << EOI
set dhcp.@dnsmasq[0].serversfile='/etc/dnsmasq.servers'
commit dhcp
EOI

# Configure timezone
uci -q batch << EOI
set system.@system[0].zonename='Asia/Shanghai'
set system.@system[0].timezone='CST-8'
commit system
EOI

# Configure tunsafe
mkdir -p /etc/tunsafe
touch /etc/tunsafe/tunsafe.conf
wget -O /usr/bin/tunsafe https://github.com/lmq8267/tunsafe/releases/download/2025-10-09/tunsafe_mipsel-linux && chmod +x /usr/bin/tunsafe

cat > /etc/init.d/tunsafe << 'EOF'
#!/bin/sh /etc/rc.common
# Copyright (C) 2025 OpenWrt.org

START=95
STOP=10
USE_PROCD=1

PROG=/usr/bin/tunsafe
CONFIG_FILE=/etc/tunsafe/tunsafe.conf
PID_FILE=/var/run/tunsafe.pid

start_service() {

    procd_open_instance
    procd_set_param command "$PROG" start "$CONFIG_FILE"

    procd_set_param respawn
    procd_set_param stdout 1
    procd_set_param stderr 1
    procd_set_param pidfile "$PID_FILE"
    procd_close_instance
}

stop_service() {
    # procd 会自动停止
    true
}

reload_service() {
    stop
    start
}
EOF

chmod +x /etc/init.d/tunsafe
/etc/init.d/tunsafe enabled

# Configure ProtonVPN interface
uci -q batch << EOI
set network.protonvpn=interface
set network.protonvpn.proto='none'
set network.protonvpn.device='tun0'
set network.protonvpn.defaultroute='0'
set network.protonvpn.multipath='off'
commit network
del firewall.@zone[1].network
add_list firewall.@zone[1].network='wan'
add_list firewall.@zone[1].network='protonvpn'
commit firewall
EOI

# Configure pbr
cat > /etc/config/pbr << 'EOF'
config pbr 'config'
        option enabled '1'
        option fw_mask '00ff0000'
        option ipv6_enabled '1'
        option nft_rule_counter '0'
        option nft_set_auto_merge '1'
        option nft_set_counter '0'
        option nft_set_flags_interval '1'
        option nft_set_flags_timeout '0'
        option nft_set_policy 'performance'
        option nft_warper_set_counter '0'
        option procd_boot_trigger_delay '5000'
        option procd_reload_delay '0'
        option resolver_set 'dnsmasq.nftset'
        option strict_enforcement '1'
        option uplink_interface 'wan'
        option uplink_interface6 'wan6'
        option uplink_ip_rules_priority '30000'
        option uplink_mark '00010000'
        option verbosity '2'
        list ignored_interface 'vpnserver'
        list lan_device 'br-lan'
        list resolver_instance '*'
        list webui_supported_protocol 'all'
        list webui_supported_protocol 'tcp'
        list webui_supported_protocol 'udp'
        list webui_supported_protocol 'tcp udp'
        list webui_supported_protocol 'icmp'
        option config_compat '25'
        option config_version '1.2.2-r14'
        option rule_create_option 'add'
        option webui_show_ignore_target '0'

config include
        option path '/warpr/share/pbr/pbr.warper.dnsprefetch'
        option enabled '0'

config include
        option path '/warpr/share/pbr/pbr.warper.aws'
        option enabled '0'

config include
        option path '/warpr/share/pbr/pbr.warper.netflix'
        option enabled '0'

config dns_policy
        option name 'Redirect Local IP DNS'
        option src_addr '192.168.1.5'
        option dest_dns '1.1.1.1'
        option dest_dns_port '53'
        option enabled '0'

config policy
        option name 'Ignore Local Requests'
        option interface 'ignore'
        option dest_addr '10.0.0.0/24 10.0.1.0/24 192.168.100.0/24 192.168.1.0/24'
        option enabled '0'

config policy
        option name 'Plex/Emby Local Server'
        option interface 'wan'
        option src_port '8096 8920 32400'
        option enabled '0'

config policy
        option name 'Plex/Emby Remote Servers'
        option interface 'wan'
        option dest_addr 'plex.tv my.plexapp.com emby.media app.emby.media tv.emby.media'
        option enabled '0'

config policy
        option name 'telegram'
        option dest_addr '91.108.56.0/22 91.108.4.0/22 91.108.8.0/22 91.108.16.0/22 91.108.12.0/22 149.154.160.0/20 91.105.192.0/23 91.108.20.0/22 185.76.151.0/24 2001:b28:f23d::/48 2001:b28:f23f::/48 2001:67c:4e8::/48 2001:b28:f23c::/48 2a0a:f280::/32 telegra.ph'
        option interface 'protonvpn'

config policy
        option name 'ai'
        option dest_addr 'mss.office.com m365.cloud.microsoft chatgpt.com openai.com oaistatic.com'
        option interface 'protonvpn'

config policy
        option name 'github'
        option dest_addr 'github.com githubusercontent.com githubassets.com'
        option interface 'protonvpn'

config policy
        option name 'google'
        option dest_addr 'google.com google.com.hk googleapis.com googlewarpercontent.com googlevideo.com youtube.com youtu.be ytimg.com ggpht.com gstatic.com'
        option interface 'protonvpn'

config policy
        option name 'x'
        option dest_addr 'twitter.com t.co twimg.com x.com'
        option interface 'protonvpn'

config policy
        option name 'facebook'
        option dest_addr 'facebook.com fb.com fbcdn.net instagram.com cdninstagram.com messenger.com'
        option interface 'protonvpn'

config policy
        option name 'proton'
        option dest_addr 'proton.me'
        option interface 'protonvpn'

config policy
        option name 'fast'
        option dest_addr 'fast.com netflix.com nflxvideo.net'
        option interface 'protonvpn'

config policy
        option name 'reddit'
        option dest_addr 'reddit.com redditmedia.com redditstatic.com redd.it'
        option interface 'protonvpn'

config policy
        option name 'steamcommunity'
        option dest_addr 'steamcommunity.com'
        option interface 'protonvpn'
EOF

echo "All done!"
```
