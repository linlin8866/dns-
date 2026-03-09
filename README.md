dns解锁
chattr -i /etc/resolv.conf
锁定
chattr +i /etc/resolv.conf
创建脚本
nano net_optimize.sh
权限运行
chmod +x net_optimize.sh
sudo ./net_optimize.sh
测试ipv6
ping6 google.com
开启bbr
curl -L https://raw.githubusercontent.com/teddysun/across/master/bbr.sh | bash
确认bbr
sysctl net.ipv4.tcp_congestion_control
lsmod | grep bbr
查看dns
cat /etc/resolv.conf
开启tcp

echo 3 > /proc/sys/net/ipv4/tcp_fastopen
永久tcp
net.ipv4.tcp_fastopen=3
查看网络信息

ip a && echo && ip route && echo && cat /etc/resolv.conf



自动获取IP脚本

#!/bin/bash

echo "======================================"
echo " VPS 网络自动配置脚本"
echo " IPv4 + IPv6 + DNS + Hosts"
echo "======================================"

HOSTNAME=$(hostname)

echo
echo "1️⃣ 获取服务器 IP"

IPV4=$(curl -4 -s https://api.ipify.org)
IPV6=$(curl -6 -s https://api64.ipify.org)

echo "IPv4: $IPV4"
echo "IPv6: $IPV6"

echo
echo "2️⃣ 设置 hosts"

cat > /etc/hosts <<EOF
127.0.0.1 localhost
$IPV4 $HOSTNAME
EOF

if [ ! -z "$IPV6" ]; then
echo "$IPV6 $HOSTNAME" >> /etc/hosts
fi

echo "hosts 更新完成"

echo
echo "3️⃣ 关闭 systemd DNS"

systemctl stop systemd-resolved 2>/dev/null
systemctl disable systemd-resolved 2>/dev/null

rm -f /etc/resolv.conf

echo
echo "4️⃣ 写入 DNS"

cat > /etc/resolv.conf <<EOF
nameserver 208.67.222.222
nameserver 208.67.220.220
nameserver 1.1.1.1
nameserver 8.8.8.8
nameserver 2606:4700:4700::1111
nameserver 2001:4860:4860::8888
EOF

echo
echo "5️⃣ 锁定 DNS"

chattr +i /etc/resolv.conf

echo
echo "当前 DNS"

cat /etc/resolv.conf

echo
echo "6️⃣ 测试 IPv4 DNS"

dig google.com A +short

echo
echo "7️⃣ 测试 IPv6 DNS"

dig google.com AAAA +short

echo
echo "8️⃣ 网络测试"

ping -4 -c 2 google.com

if ping -6 -c 2 google.com >/dev/null 2>&1
then
echo "IPv6 网络正常"
else
echo "IPv6 网络不可用"
fi

echo
echo "======================================"
echo " VPS 网络配置完成 ✔"
echo "======================================"
