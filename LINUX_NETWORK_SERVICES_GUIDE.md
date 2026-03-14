# Linux 网络服务学习指南

## 目录

1. [网络基础概念](#网络基础概念)
2. [网络配置与管理](#网络配置与管理)
3. [常用网络服务](#常用网络服务)
4. [网络安全与防火墙](#网络安全与防火墙)
5. [网络监控与故障排查](#网络监控与故障排查)
6. [高级网络服务](#高级网络服务)
7. [实战案例](#实战案例)

---

## 网络基础概念

### OSI 七层模型

| 层次 | 名称 | 功能 | 协议示例 |
|------|------|------|----------|
| 7 | 应用层 | 为应用程序提供网络服务 | HTTP, FTP, SMTP, DNS |
| 6 | 表示层 | 数据格式转换、加密解密 | SSL/TLS, JPEG, MPEG |
| 5 | 会话层 | 建立、维护、终止会话 | NetBIOS, RPC |
| 4 | 传输层 | 端到端可靠传输 | TCP, UDP |
| 3 | 网络层 | 路由选择、逻辑寻址 | IP, ICMP, OSPF |
| 2 | 数据链路层 | 物理寻址、帧封装 | Ethernet, PPP, HDLC |
| 1 | 物理层 | 比特流传输 | RS-232, RJ45, 光纤 |

### TCP/IP 协议栈

```
应用层：HTTP, HTTPS, FTP, SSH, DNS, SMTP, POP3, IMAP
传输层：TCP (可靠), UDP (快速)
网络层：IP, ICMP, IGMP
链路层：Ethernet, Wi-Fi, PPP
```

### 重要概念

#### IP 地址
- **IPv4**: 32位，如 192.168.1.100
- **IPv6**: 128位，如 2001:0db8:85a3:0000:0000:8a2e:0370:7334
- **私有地址范围**:
  - A类: 10.0.0.0/8
  - B类: 172.16.0.0/12
  - C类: 192.168.0.0/16

#### 子网掩码
- 决定网络部分和主机部分
- CIDR 表示法: 192.168.1.0/24

#### 端口号
- 知名端口: 0-1023 (需要 root 权限)
- 注册端口: 1024-49151
- 动态端口: 49152-65535

常用端口:
```
20/21: FTP
22: SSH
23: Telnet
25: SMTP
53: DNS
80: HTTP
110: POP3
143: IMAP
443: HTTPS
993: IMAPS
995: POP3S
3306: MySQL
5432: PostgreSQL
6379: Redis
8080: HTTP Proxy/Alt HTTP
```

---

## 网络配置与管理

### 网络接口配置

#### 1. 使用 ip 命令（推荐）

```bash
# 查看网络接口
ip addr show
ip a

# 查看特定接口
ip addr show eth0

# 启用/禁用接口
ip link set eth0 up
ip link set eth0 down

# 配置 IP 地址
ip addr add 192.168.1.100/24 dev eth0
ip addr del 192.168.1.100/24 dev eth0

# 添加多个 IP 地址
ip addr add 192.168.1.101/24 dev eth0

# 查看路由表
ip route show
ip r

# 添加默认网关
ip route add default via 192.168.1.1

# 添加静态路由
ip route add 10.0.0.0/8 via 192.168.1.254

# 删除路由
ip route del 10.0.0.0/8
```

#### 2. 使用传统命令（ifconfig, route）

```bash
# 查看网络接口
ifconfig
ifconfig eth0

# 配置 IP 地址
ifconfig eth0 192.168.1.100 netmask 255.255.255.0

# 启用/禁用接口
ifconfig eth0 up
ifconfig eth0 down

# 添加默认网关
route add default gw 192.168.1.1

# 查看路由表
route -n
```

#### 3. 配置文件方式

**Ubuntu/Debian - Netplan (/etc/netplan/*.yaml)**

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: no
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
```

应用配置:
```bash
sudo netplan apply
```

**CentOS/RHEL - ifcfg 文件 (/etc/sysconfig/network-scripts/ifcfg-eth0)**

```bash
TYPE=Ethernet
BOOTPROTO=static
NAME=eth0
DEVICE=eth0
ONBOOT=yes
IPADDR=192.168.1.100
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
DNS1=8.8.8.8
DNS2=8.8.4.4
```

重启网络服务:
```bash
sudo systemctl restart network
# 或
sudo nmcli connection reload
sudo nmcli connection up eth0
```

### DNS 配置

```bash
# 查看 DNS 配置
cat /etc/resolv.conf

# 编辑 DNS 配置
sudo vim /etc/resolv.conf

# 添加 DNS 服务器
nameserver 8.8.8.8
nameserver 8.8.4.4
search example.com

# 测试 DNS 解析
nslookup google.com
dig google.com
host google.com
```

### 主机名配置

```bash
# 查看主机名
hostname
hostnamectl

# 临时修改主机名
sudo hostname new-hostname

# 永久修改主机名
sudo hostnamectl set-hostname new-hostname

# 编辑 hosts 文件
sudo vim /etc/hosts
# 添加:
# 192.168.1.100  myserver.example.com myserver
```

---

## 常用网络服务

### 1. SSH 服务 (OpenSSH)

#### 安装与配置

```bash
# 安装 OpenSSH
sudo apt install openssh-server  # Ubuntu/Debian
sudo yum install openssh-server  # CentOS/RHEL

# 启动服务
sudo systemctl start sshd
sudo systemctl enable sshd
sudo systemctl status sshd

# 配置文件
sudo vim /etc/ssh/sshd_config
```

#### 重要配置项

```bash
# /etc/ssh/sshd_config

# 监听端口
Port 22

# 监听地址
ListenAddress 0.0.0.0

# 禁止 root 登录
PermitRootLogin no

# 使用密钥认证
PubkeyAuthentication yes

# 禁用密码认证（推荐）
PasswordAuthentication no

# 允许的用户
AllowUsers user1 user2

# 最大认证尝试次数
MaxAuthTries 3

# 空闲超时
ClientAliveInterval 300
ClientAliveCountMax 2
```

#### SSH 密钥认证

```bash
# 生成密钥对
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
ssh-keygen -t ed25519 -C "your_email@example.com"  # 推荐

# 复制公钥到远程服务器
ssh-copy-id user@remote-host

# 手动复制公钥
cat ~/.ssh/id_rsa.pub | ssh user@remote-host "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

# 设置权限
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

#### SSH 客户端使用

```bash
# 连接远程服务器
ssh user@remote-host
ssh -p 2222 user@remote-host  # 指定端口

# 使用密钥文件
ssh -i ~/.ssh/mykey user@remote-host

# SSH 配置文件 (~/.ssh/config)
Host myserver
    HostName 192.168.1.100
    User myuser
    Port 22
    IdentityFile ~/.ssh/id_rsa

# 使用别名连接
ssh myserver

# SSH 隧道/端口转发
# 本地端口转发
ssh -L 8080:remote-server:80 user@ssh-server

# 远程端口转发
ssh -R 8080:local-server:80 user@ssh-server

# 动态端口转发 (SOCKS 代理)
ssh -D 1080 user@ssh-server

# SCP 文件传输
scp file.txt user@remote-host:/path/to/destination
scp -r directory/ user@remote-host:/path/to/destination

# SFTP 文件传输
sftp user@remote-host
```

### 2. Web 服务器

#### Nginx

```bash
# 安装 Nginx
sudo apt install nginx  # Ubuntu/Debian
sudo yum install nginx  # CentOS/RHEL

# 服务管理
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl reload nginx  # 平滑重载配置
sudo systemctl enable nginx
sudo systemctl status nginx

# 测试配置
sudo nginx -t

# 配置文件位置
/etc/nginx/nginx.conf          # 主配置文件
/etc/nginx/sites-available/    # 站点配置（Ubuntu）
/etc/nginx/sites-enabled/      # 启用的站点
/etc/nginx/conf.d/             # 额外配置
```

**基本配置示例**

```nginx
# /etc/nginx/sites-available/mysite

server {
    listen 80;
    server_name example.com www.example.com;

    root /var/www/html;
    index index.html index.htm;

    # 日志文件
    access_log /var/log/nginx/mysite_access.log;
    error_log /var/log/nginx/mysite_error.log;

    location / {
        try_files $uri $uri/ =404;
    }

    # 静态文件缓存
    location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    # 反向代理
    location /api/ {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # PHP 处理
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
    }

    # 禁止访问隐藏文件
    location ~ /\. {
        deny all;
    }
}
```

**HTTPS 配置（使用 Let's Encrypt）**

```bash
# 安装 Certbot
sudo apt install certbot python3-certbot-nginx

# 获取证书
sudo certbot --nginx -d example.com -d www.example.com

# 自动续期测试
sudo certbot renew --dry-run
```

**负载均衡配置**

```nginx
upstream backend {
    least_conn;  # 最少连接算法
    server backend1.example.com weight=5;
    server backend2.example.com;
    server backend3.example.com backup;
}

server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

#### Apache HTTP Server

```bash
# 安装 Apache
sudo apt install apache2  # Ubuntu/Debian
sudo yum install httpd    # CentOS/RHEL

# 服务管理
sudo systemctl start apache2
sudo systemctl stop apache2
sudo systemctl restart apache2
sudo systemctl reload apache2
sudo systemctl enable apache2

# 配置文件
/etc/apache2/apache2.conf      # Ubuntu/Debian
/etc/httpd/conf/httpd.conf     # CentOS/RHEL
/etc/apache2/sites-available/  # 虚拟主机配置
```

**虚拟主机配置**

```apache
# /etc/apache2/sites-available/mysite.conf

<VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com
    ServerAdmin webmaster@example.com

    DocumentRoot /var/www/mysite

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    <Directory /var/www/mysite>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

启用站点和模块:
```bash
sudo a2ensite mysite
sudo a2dissite 000-default
sudo a2enmod rewrite
sudo a2enmod ssl
sudo systemctl reload apache2
```

### 3. DNS 服务器 (BIND9)

```bash
# 安装 BIND9
sudo apt install bind9 bind9utils bind9-doc

# 配置文件
/etc/bind/named.conf.options    # 全局选项
/etc/bind/named.conf.local     # 本地区域配置
/etc/bind/named.conf.default-zones  # 默认区域

# 服务管理
sudo systemctl start named
sudo systemctl enable named
sudo systemctl status named
```

**配置示例**

```bash
# /etc/bind/named.conf.options
options {
    directory "/var/cache/bind";

    // 转发器（上游 DNS）
    forwarders {
        8.8.8.8;
        8.8.4.4;
    };

    // 允许查询的客户端
    allow-query {
        192.168.1.0/24;
        localhost;
    };

    // 递归查询
    recursion yes;

    // DNSSEC
    dnssec-validation auto;
};
```

```bash
# /etc/bind/named.conf.local
zone "example.com" {
    type master;
    file "/etc/bind/zones/db.example.com";
};

zone "1.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/zones/db.192.168.1";
};
```

**正向区域文件**

```bash
# /etc/bind/zones/db.example.com
$TTL    604800
@   IN  SOA ns1.example.com. admin.example.com. (
              2         ; Serial
         604800         ; Refresh
          86400         ; Retry
        2419200         ; Expire
         604800 )       ; Negative Cache TTL

; 名称服务器
@   IN  NS      ns1.example.com.

; A 记录
@       IN  A       192.168.1.10
ns1     IN  A       192.168.1.10
www     IN  A       192.168.1.20
mail    IN  A       192.168.1.30

; MX 记录
@   IN  MX  10  mail.example.com.

; CNAME 记录
ftp     IN  CNAME   www.example.com.
```

**反向区域文件**

```bash
# /etc/bind/zones/db.192.168.1
$TTL    604800
@   IN  SOA ns1.example.com. admin.example.com. (
              2         ; Serial
         604800         ; Refresh
          86400         ; Retry
        2419200         ; Expire
         604800 )       ; Negative Cache TTL

; 名称服务器
@   IN  NS      ns1.example.com.

; PTR 记录
10  IN  PTR     ns1.example.com.
20  IN  PTR     www.example.com.
30  IN  PTR     mail.example.com.
```

验证配置并重启:
```bash
sudo named-checkconf
sudo named-checkzone example.com /etc/bind/zones/db.example.com
sudo systemctl restart named
```

### 4. DHCP 服务器

```bash
# 安装 DHCP 服务器
sudo apt install isc-dhcp-server

# 配置文件
sudo vim /etc/dhcp/dhcpd.conf
```

**配置示例**

```bash
# /etc/dhcp/dhcpd.conf

# 全局设置
authoritative;
default-lease-time 600;
max-lease-time 7200;
log-facility local7;

# DNS 服务器
option domain-name "example.com";
option domain-name-servers ns1.example.com, ns2.example.com;

# 子网配置
subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.100 192.168.1.200;
    option routers 192.168.1.1;
    option subnet-mask 255.255.255.0;
    option broadcast-address 192.168.1.255;
    option domain-name-servers 8.8.8.8, 8.8.4.4;
    default-lease-time 3600;
    max-lease-time 86400;
}

# 静态 IP 分配（基于 MAC 地址）
host printer {
    hardware ethernet 00:11:22:33:44:55;
    fixed-address 192.168.1.50;
}

host server1 {
    hardware ethernet AA:BB:CC:DD:EE:FF;
    fixed-address 192.168.1.10;
}
```

启动服务:
```bash
sudo systemctl start isc-dhcp-server
sudo systemctl enable isc-dhcp-server
sudo systemctl status isc-dhcp-server
```

### 5. FTP 服务器 (vsftpd)

```bash
# 安装 vsftpd
sudo apt install vsftpd

# 配置文件
sudo vim /etc/vsftpd.conf
```

**配置示例**

```bash
# /etc/vsftpd.conf

# 基本设置
listen=NO
listen_ipv6=YES
anonymous_enable=NO
local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
use_localtime=YES
xferlog_enable=YES
connect_from_port_20=YES

# chroot 设置（限制用户在主目录）
chroot_local_user=YES
chroot_list_enable=YES
chroot_list_file=/etc/vsftpd.chroot_list
allow_writeable_chroot=YES

# 被动模式
pasv_enable=YES
pasv_min_port=40000
pasv_max_port=50000

# SSL/TLS 加密
ssl_enable=YES
rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
ssl_tlsv1=YES
ssl_sslv2=NO
ssl_sslv3=NO
require_ssl_reuse=NO
ssl_ciphers=HIGH
```

管理用户:
```bash
# 创建 FTP 用户
sudo useradd -m -s /bin/bash ftpuser
sudo passwd ftpuser

# 添加到 chroot 列表（允许访问上级目录）
echo "ftpuser" | sudo tee -a /etc/vsftpd.chroot_list
```

重启服务:
```bash
sudo systemctl restart vsftpd
sudo systemctl enable vsftpd
```

### 6. 邮件服务器 (Postfix + Dovecot)

#### Postfix (SMTP 服务器)

```bash
# 安装 Postfix
sudo apt install postfix

# 配置文件
sudo vim /etc/postfix/main.cf
```

**基本配置**

```bash
# /etc/postfix/main.cf

smtpd_banner = $myhostname ESMTP $mail_name (Ubuntu)
biff = no

# 附加域名
append_dot_mydomain = no

# TLS 参数
smtpd_tls_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
smtpd_tls_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
smtpd_tls_security_level=may
smtp_tls_security_level=may
smtpd_tls_session_cache_database = btree:${data_directory}/smtpd_scache
smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache

# 邮箱大小限制
mailbox_size_limit = 0
recipient_delimiter = +
inet_interfaces = all
inet_protocols = all

# 基本配置
myhostname = mail.example.com
mydomain = example.com
myorigin = $mydomain
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128
home_mailbox = Maildir/

# SMTP 认证
smtpd_sasl_auth_enable = yes
smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth
```

#### Dovecot (IMAP/POP3 服务器)

```bash
# 安装 Dovecot
sudo apt install dovecot-core dovecot-imapd dovecot-pop3d

# 配置文件
sudo vim /etc/dovecot/dovecot.conf
sudo vim /etc/dovecot/conf.d/10-mail.conf
sudo vim /etc/dovecot/conf.d/10-auth.conf
sudo vim /etc/dovecot/conf.d/10-master.conf
```

**配置示例**

```bash
# /etc/dovecot/conf.d/10-mail.conf
mail_location = maildir:~/Maildir

# /etc/dovecot/conf.d/10-auth.conf
auth_mechanisms = plain login

# /etc/dovecot/conf.d/10-master.conf
service auth {
    unix_listener /var/spool/postfix/private/auth {
        mode = 0666
        user = postfix
        group = postfix
    }
}
```

重启服务:
```bash
sudo systemctl restart postfix
sudo systemctl restart dovecot
sudo systemctl enable postfix dovecot
```

---

## 网络安全与防火墙

### iptables

```bash
# 查看规则
sudo iptables -L
sudo iptables -L -n -v
sudo iptables -L -t nat

# 基本规则
# 允许已建立的连接
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# 允许本地回环
sudo iptables -A INPUT -i lo -j ACCEPT

# 允许 SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# 允许 HTTP/HTTPS
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# 允许特定 IP
sudo iptables -A INPUT -s 192.168.1.100 -j ACCEPT

# 拒绝其他所有输入
sudo iptables -A INPUT -j DROP

# 删除规则
sudo iptables -D INPUT 1

# 清空所有规则
sudo iptables -F

# 保存规则
sudo iptables-save > /etc/iptables/rules.v4
sudo iptables-restore < /etc/iptables/rules.v4
```

### UFW (Uncomplicated Firewall)

```bash
# 安装 UFW
sudo apt install ufw

# 查看状态
sudo ufw status verbose

# 设置默认策略
sudo ufw default deny incoming
sudo ufw default allow outgoing

# 允许服务
sudo ufw allow ssh
sudo ufw allow 22
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# 允许端口范围
sudo ufw allow 6000:6007/tcp

# 允许特定 IP
sudo ufw allow from 192.168.1.100
sudo ufw allow from 192.168.1.0/24 to any port 22

# 拒绝访问
sudo ufw deny 23

# 删除规则
sudo ufw delete allow 80
sudo ufw status numbered
sudo ufw delete 2

# 启用/禁用
sudo ufw enable
sudo ufw disable

# 重置
sudo ufw reset
```

### firewalld (CentOS/RHEL)

```bash
# 安装
sudo yum install firewalld

# 服务管理
sudo systemctl start firewalld
sudo systemctl enable firewalld
sudo systemctl status firewalld

# 查看状态
sudo firewall-cmd --state
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --list-all

# 添加服务
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --add-service=ssh

# 添加端口
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --permanent --add-port=53/udp

# 移除服务/端口
sudo firewall-cmd --permanent --remove-service=http
sudo firewall-cmd --permanent --remove-port=8080/tcp

# 重新加载配置
sudo firewall-cmd --reload

# 端口转发
sudo firewall-cmd --permanent --add-forward-port=port=80:proto=tcp:toport=8080
```

### fail2ban (入侵防护)

```bash
# 安装
sudo apt install fail2ban

# 配置文件
sudo vim /etc/fail2ban/jail.local
```

**配置示例**

```bash
# /etc/fail2ban/jail.local

[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 5
ignoreip = 127.0.0.1/8 192.168.1.0/24

[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 86400

[nginx-http-auth]
enabled = true
filter = nginx-http-auth
port = http,https
logpath = /var/log/nginx/*error.log

[postfix]
enabled = true
port = smtp,ssmtp
filter = postfix
logpath = /var/log/mail.log
```

管理服务:
```bash
sudo systemctl restart fail2ban
sudo systemctl enable fail2ban

# 查看状态
sudo fail2ban-client status
sudo fail2ban-client status sshd

# 手动解封 IP
sudo fail2ban-client set sshd unbanip 192.168.1.100
```

---

## 网络监控与故障排查

### 网络诊断工具

#### 1. 连通性测试

```bash
# ping - 测试连通性
ping google.com
ping -c 4 192.168.1.1  # 发送 4 个包
ping -i 0.5 192.168.1.1  # 间隔 0.5 秒

# traceroute - 跟踪路由
traceroute google.com
traceroute -I google.com  # 使用 ICMP

# tracepath - 类似 traceroute
tracepath google.com

# mtr - 结合 ping 和 traceroute
mtr google.com
mtr -r -c 10 google.com  # 报告模式
```

#### 2. 端口扫描

```bash
# netstat - 网络统计
netstat -tuln  # 监听的 TCP/UDP 端口
netstat -tun  # 活动的 TCP/UDP 连接
netstat -rn  # 路由表
netstat -anp  # 所有连接及进程

# ss - socket 统计（替代 netstat）
ss -tuln
ss -tun
ss -s  # 统计摘要
ss -p  # 显示进程

# nmap - 端口扫描
nmap 192.168.1.1
nmap -p 22,80,443 192.168.1.1
nmap -sV 192.168.1.1  # 版本检测
nmap -O 192.168.1.1  # 操作系统检测
nmap -sS 192.168.1.0/24  # SYN 扫描

# nc (netcat) - 端口测试
nc -zv 192.168.1.1 22
nc -zv 192.168.1.1 20-80
nc -l 1234  # 监听端口
nc 192.168.1.1 1234  # 连接端口
```

#### 3. 网络流量分析

```bash
# tcpdump - 抓包工具
tcpdump -i eth0
tcpdump -i eth0 port 80
tcpdump -i eth0 host 192.168.1.100
tcpdump -i eth0 -w capture.pcap  # 保存到文件
tcpdump -r capture.pcap  # 读取文件
tcpdump -i eth0 -nn -S 'tcp port 80'

# wireshark - 图形化抓包工具
sudo apt install wireshark
sudo wireshark

# iftop - 实时带宽监控
sudo apt install iftop
sudo iftop -i eth0

# nethogs - 按进程监控带宽
sudo apt install nethogs
sudo nethogs eth0

# iptraf - 交互式网络监控
sudo apt install iptraf
sudo iptraf
```

#### 4. DNS 诊断

```bash
# nslookup
nslookup google.com
nslookup -type=mx google.com

# dig - DNS 查询工具
dig google.com
dig @8.8.8.8 google.com
dig +short google.com
dig -x 192.168.1.1  # 反向查询

# host
host google.com
host -t mx google.com
```

#### 5. 网络性能测试

```bash
# iperf3 - 网络带宽测试
# 服务端
iperf3 -s

# 客户端
iperf3 -c server-ip
iperf3 -c server-ip -t 60  # 测试 60 秒
iperf3 -c server-ip -P 10  # 10 个并行流

# curl - HTTP 测试
curl -I http://example.com
curl -v http://example.com
curl -o /dev/null -s -w "Time: %{time_total}s\n" http://example.com

# wget - 下载测试
wget http://example.com/file.zip
wget -O /dev/null http://example.com/file.zip
```

### 日志分析

```bash
# 系统日志
/var/log/syslog          # Ubuntu/Debian
/var/log/messages        # CentOS/RHEL
/var/log/auth.log        # 认证日志
/var/log/kern.log        # 内核日志

# 网络服务日志
/var/log/nginx/access.log
/var/log/nginx/error.log
/var/log/apache2/access.log
/var/log/apache2/error.log
/var/log/mail.log
/var/log/mysql/error.log

# 查看日志
tail -f /var/log/syslog
less /var/log/nginx/access.log
grep "error" /var/log/syslog
journalctl -u nginx -f
```

---

## 高级网络服务

### 1. VPN 服务器 (OpenVPN)

```bash
# 安装 OpenVPN
sudo apt install openvpn easy-rsa

# 设置 CA 和证书
make-cadir ~/openvpn-ca
cd ~/openvpn-ca
source vars
./clean-all
./build-ca
./build-key-server server
./build-dh
openvpn --genkey --secret keys/ta.key

# 生成客户端证书
./build-key client1

# 配置服务器
sudo cp ~/openvpn-ca/keys/{server.crt,server.key,ca.crt,dh2048.pem,ta.key} /etc/openvpn/
sudo cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz /etc/openvpn/
sudo gzip -d /etc/openvpn/server.conf.gz
```

**服务器配置示例**

```bash
# /etc/openvpn/server.conf
port 1194
proto udp
dev tun
ca ca.crt
cert server.crt
key server.key
dh dh2048.pem
server 10.8.0.0 255.255.255.0
ifconfig-pool-persist ipp.txt
push "redirect-gateway def1 bypass-dhcp"
push "dhcp-option DNS 8.8.8.8"
push "dhcp-option DNS 8.8.4.4"
keepalive 10 120
tls-auth ta.key 0
cipher AES-256-CBC
user nobody
group nogroup
persist-key
persist-tun
status openvpn-status.log
verb 3
```

启动服务:
```bash
sudo systemctl start openvpn@server
sudo systemctl enable openvpn@server
```

### 2. 代理服务器 (Squid)

```bash
# 安装 Squid
sudo apt install squid

# 配置文件
sudo vim /etc/squid/squid.conf
```

**配置示例**

```bash
# /etc/squid/squid.conf

# 监听端口
http_port 3128

# 缓存目录
cache_dir ufs /var/spool/squid 100 16 256

# 访问控制
acl localnet src 192.168.1.0/24
acl SSL_ports port 443
acl Safe_ports port 80 21 443 1025-65535

http_access deny !Safe_ports
http_access deny CONNECT !SSL_ports
http_access allow localnet
http_access allow localhost
http_access deny all

# 缓存设置
cache_mem 256 MB
maximum_object_size_in_memory 512 KB
maximum_object_size 4 MB

# 日志
access_log /var/log/squid/access.log squid
cache_log /var/log/squid/cache.log
```

重启服务:
```bash
sudo systemctl restart squid
sudo systemctl enable squid
```

### 3. 负载均衡器 (HAProxy)

```bash
# 安装 HAProxy
sudo apt install haproxy

# 配置文件
sudo vim /etc/haproxy/haproxy.cfg
```

**配置示例**

```bash
# /etc/haproxy/haproxy.cfg

global
    log /dev/log local0
    log /dev/log local1 notice
    chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin
    stats timeout 30s
    user haproxy
    group haproxy
    daemon

defaults
    log     global
    mode    http
    option  httplog
    option  dontlognull
    timeout connect 5000
    timeout client  50000
    timeout server  50000

frontend http_front
    bind *:80
    stats uri /haproxy?stats
    default_backend http_back

backend http_back
    balance roundrobin
    server backend1 192.168.1.101:80 check
    server backend2 192.168.1.102:80 check
    server backend3 192.168.1.103:80 check
```

重启服务:
```bash
sudo systemctl restart haproxy
sudo systemctl enable haproxy
```

---

## 实战案例

### 案例 1: 搭建完整的 Web 服务器环境

```bash
# 1. 更新系统
sudo apt update && sudo apt upgrade -y

# 2. 安装必要软件
sudo apt install -y nginx mysql-server php-fpm php-mysql

# 3. 配置 Nginx
sudo vim /etc/nginx/sites-available/webapp

# 4. 配置 PHP
sudo vim /etc/php/7.4/fpm/php.ini

# 5. 配置 MySQL
sudo mysql_secure_installation

# 6. 配置防火墙
sudo ufw allow 'Nginx Full'
sudo ufw allow ssh
sudo ufw enable

# 7. 启动服务
sudo systemctl restart nginx
sudo systemctl restart php7.4-fpm
sudo systemctl restart mysql

# 8. 测试
curl http://localhost
```

### 案例 2: 配置内网 DNS 服务器

```bash
# 1. 安装 BIND9
sudo apt install -y bind9 bind9utils bind9-doc

# 2. 配置选项
sudo vim /etc/bind/named.conf.options

# 3. 创建区域文件
sudo mkdir /etc/bind/zones
sudo vim /etc/bind/zones/db.company.local

# 4. 配置区域
sudo vim /etc/bind/named.conf.local

# 5. 验证配置
sudo named-checkconf
sudo named-checkzone company.local /etc/bind/zones/db.company.local

# 6. 重启服务
sudo systemctl restart named

# 7. 测试
dig @localhost server1.company.local
```

### 案例 3: 搭建监控服务器

```bash
# 1. 安装 Prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.30.0/prometheus-2.30.0.linux-amd64.tar.gz
tar xvfz prometheus-*.tar.gz
cd prometheus-*

# 2. 配置 Prometheus
vim prometheus.yml

# 3. 启动 Prometheus
./prometheus --config.file=prometheus.yml

# 4. 安装 Grafana
sudo apt install -y grafana
sudo systemctl start grafana-server
sudo systemctl enable grafana-server

# 5. 访问 Grafana
# http://localhost:3000 (admin/admin)

# 6. 添加 Prometheus 数据源
# 在 Grafana UI 中配置
```

---

## 学习路径建议

### 初级阶段
1. 掌握网络基础概念（OSI 模型、TCP/IP）
2. 学习基本网络配置命令（ip, ifconfig, route）
3. 配置静态 IP、DNS、网关
4. 使用基本诊断工具（ping, traceroute, netstat）

### 中级阶段
1. 配置和管理常用网络服务（SSH, Web, DNS, DHCP）
2. 学习防火墙配置（iptables, UFW, firewalld）
3. 掌握网络监控工具（tcpdump, nmap, ss）
4. 理解网络安全基础（fail2ban, SSL/TLS）

### 高级阶段
1. 配置复杂网络服务（VPN, 代理, 负载均衡）
2. 网络性能优化和调优
3. 高可用和容错配置
4. 自动化运维（Ansible, Puppet）

---

## 参考资源

### 官方文档
- [Linux Network Administrator's Guide](http://www.tldp.org/LDP/nag2/)
- [Ubuntu Networking Documentation](https://ubuntu.com/server/docs/networking)
- [Red Hat Networking Guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_and_managing_networking)

### 在线学习
- [Linux Foundation Networking Courses](https://training.linuxfoundation.org/)
- [Cybrary Linux Network Services](https://www.cybrary.it/)
- [Linux Journey - Networking](https://linuxjourney.com/lesson/networking)

### 工具文档
- [Nginx Documentation](https://nginx.org/en/docs/)
- [BIND 9 Administrator Reference Manual](https://bind9.readthedocs.io/)
- [Postfix Documentation](http://www.postfix.org/documentation.html)
- [Wireshark User's Guide](https://www.wireshark.org/docs/wsug_html_chunked/)

### 社区资源
- [Server Fault](https://serverfault.com/) - 系统管理问答
- [Unix & Linux Stack Exchange](https://unix.stackexchange.com/)
- [Reddit r/linuxadmin](https://www.reddit.com/r/linuxadmin/)

---

## 总结

Linux 网络服务是系统管理员的核心技能。通过本指南，您应该能够：

1. 理解网络基础概念和协议
2. 配置和管理网络接口
3. 部署和维护常用网络服务
4. 实施网络安全措施
5. 使用监控和诊断工具
6. 搭建高级网络服务

持续实践和深入学习是掌握 Linux 网络服务的关键。建议在虚拟机或云环境中搭建测试环境，逐步实践各种网络服务的配置和管理。
