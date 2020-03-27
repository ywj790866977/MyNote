



# 初始化脚本

```shell
#!/bin/bash

# 安装epel源和常用软件

yum install -y epel-release
yum install vim lsof wget iftop lrzsz unzip pcre-devel openssh* telnet bash-completion rsync ntpdate gcc automake autoconf libtool make gcc gcc-c++ openssl openssl-devel pymongo openssl openssl-devel pymongo subversion  apr apr-util numactl gperftools libunwind libselinux-python patch  apr-devel iptables-services ntpdate zabbix30-agent -y

# 关闭selinux

sed -i "s/SELINUX=enforcing/SELINUX=disabled/g"      /etc/selinux/config


# 修改系统文件限制

cat >> /etc/security/limits.conf <<EOF

* soft nproc 8192
* hard nproc 8192
* soft nofile 8192
* hard nofile 8192
  EOF

# 关闭firewalld

systemctl stop firewalld  && systemctl disable firewalld

# 设置内核参数

/usr/bin/cp -f /etc/sysctl.conf /etc/sysctl.conf.bak
cat > /etc/sysctl.conf << EOF
vm.swappiness = 0
vm.dirty_ratio = 80
vm.dirty_background_ratio = 5
vm.dirty_ratio = 80
vm.dirty_background_ratio = 5
fs.file-max = 2097152
fs.suid_dumpable = 0
net.core.somaxconn = 65535
net.core.netdev_max_backlog = 262144
net.core.optmem_max = 25165824
net.core.rmem_default = 31457280
net.core.rmem_max = 67108864
net.core.wmem_default = 31457280
net.ipv4.tcp_syncookies = 1
net.ipv4.conf.all.rp_filter = 1
net.ipv4.icmp_echo_ignore_all = 1
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.conf.all.log_martians = 1
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv6.conf.all.disable_ipv6 =1
net.ipv6.conf.default.disable_ipv6 =1
EOF

/usr/sbin/sysctl -p

# 关闭邮件服务

systemctl stop postfix && systemctl enable postfix


# 关闭 networkmanager

systemctl stop NetworkManager > /dev/null 2>&1
systemctl disable NetworkManager > /dev/null 2>&1


# 开启iptables

echo "

# Firewall configuration written by system-config-firewall

# Manual customization of this file is not recommended.

*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 443 -j ACCEPT
-A OUTPUT  -j ACCEPT

-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
COMMIT
" > /etc/sysconfig/iptables

systemctl restart iptables
systemctl enable iptables
```



