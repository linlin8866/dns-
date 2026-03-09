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

