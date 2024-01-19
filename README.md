# You Shall Not Pass 🛡️

## Introduction 📖

The aim of this project is to set up a virtual network infrastructure with several virtual machines with specific roles.
To make this possible, we needed to realise the following steps:

- Gateway configuration
- Packet Filtering
- DHCP server configuration
- Network interface configuration

The main goal is to have all virtual machines able to communicate with each other

## VM1 (Gateway)

### Install

ISO OpenBSD 7.4
<https://cdn.openbsd.org/pub/OpenBSD/7.4/amd64/install74.iso>

VM1 contains 4 networks cards

- 1 bridge or nat
- 3 private networks

### Config

#### - DHCP

Create the following files and fill them with the following data:

- I:

```bash
/etc/dhcpd.conf
```

```bash
#       $OpenBSD: dhcpd.conf,v 1.1 2014/07/11 21:20:10 deraadt Exp $
#
# DHCP server options.
# See dhcpd.conf(5) and dhcpd(8) for more information.
#

# Network:              192.168.1.0/255.255.255.0
# Domain name:          my.domain
# Name servers:         192.168.1.3 and 192.168.1.5
# Default router:       192.168.1.1
# Addresses:            192.168.1.32 - 192.168.1.127
#
# option  domain-name "my.domain";
# option  domain-name-servers 192.168.1.3, 192.168.1.5;

# lan-1 administration

subnet 192.168.42.0 netmask 255.255.255.192 {
   option routers 192.168.42.1;
   range 192.168.42.40 192.168.42.60;
}

# lan-2 server

subnet 192.168.42.64 netmask 255.255.255.192 {
   option routers 192.168.42.65;
   range 192.168.42.70 192.168.42.110;

   host static-client {
      hardware ethernet 08:00:27:17:DC:33;
      fixed-address 192.168.42.70;
   }
}

# lan-3 employee

subnet 192.168.42.128 netmask 255.255.255.192 {
   option routers 192.168.42.129;
   range 192.168.42.140 192.168.42.180;
} 
```

- II :

```bash
/etc/hostname.em1 
```

```bash
inet 192.168.42.1 255.255.255.192 
```

- III :

```bash
 /etc/hostname.em2 
  ```

```bash
net 192.168.42.65 255.255.255.192 
```

- IV :

```bash
 /etc/hostname.em3
  ```

 ```bash
inet 192.168.42.129 255.255.255.192
 ```

- Start the DHCP service

```bash
 rcctl enable dhcpd
  ```

```bash
 tcctl start dhcpd
  ```

#### - Security

Create the following file and fill with the following data:

- I:

```bash
 /etc/pf.conf
 ```

```bash
ext_if = "em0"
ADMIN = "em1"
SERVER = "em2"
EMPLOYEE = "em3"
NETWORKINTERFACES = "{ em1, em2, em3 }"
NETWORKS = "{ em1:network, em2:network, em3:network }"
webserver = 192.168.42.70
set skip on lo
block all
pass out on $ext_if
pass in on $ext_if proto tcp from any to any port 22

# Ping
pass in on $NETWORKINTERFACES proto icmp from $NETWORKS
pass out on $NETWORKINTERFACES proto icmp from $NETWORKS

#Web Server
pass in on $NETWORKINTERFACES proto { udp tcp } from any to $webserver port { 80 443 }

# DNS Rules
pass in on $NETWORKINTERFACES proto { udp tcp } from any to any port { 53 80 443 }
pass out on $NETWORKINTERFACES proto { udp tcp } from any to any port { 53 80 443 }

# LAN to WAN by gateway
pass out on $ext_if from $NETWORKS to any nat-to $ext_if
```

- II:

```bash
 pfctl -f /etc/pf.conf
  ```

- III:

```bash
 pfctl -e
  ```

#### - Internet

Go to

```bash
/etc/syscl.conf
```

And put in

```bash
net.inet.ip.forwarding=1
```

Congratulations, you just have configured the first VM !

## VM2 (Web Server)

### Install

ISO FreeBSD 13.2
<https://download.freebsd.org/ftp/releases/ISO-IMAGES/13.2/FreeBSD-13.2-RELEASE-amd64-bootonly.iso>

### Config

#### - Nginx

I:

```bash
pkg install nginx
```

II:

```bash
service nginx enable
```

III:

```bash
service nginx start
```

#### - Php 8.1

I:

```bash
pkg install php81
```

II:

```bash
cp -v /usr/local/etc/php.ini-production /usr/local/etc/php.ini
```

III:

```bash
pkg install vim php81-xml mod_php81 php81-zip php81-mbstring php81-zlib php81-curl php81-mysqli php81-gd php81-gd
```

IV:

```bash
service php-fpm enable
```

```bash
service php-fpm start
```

#### - MySQL

I:

```bash
pkg install mysql180-server
```

II:

```bash
service mysql-server onestart
```

III: 

```bash
mysql_secure_installation
```

IV:

```bash
service mysql-server enable
```

```bash
service mysql-server start
```

V: Update /etc/rc.conf with those datas : 

```bash
defaultrouter="192.168.42.65" (ip de la gateway)
mysql_enable=”YES”
nginx_enable=”YES”
php_fpm_enable=”YES”```

VI: Update /usr/local/etc/nginx/nginx.conf with those datas: 
```server {
    listen       80;
    server_name  localhost;

    location / {
        root   /usr/local/www/nginx;
        index  index.php index.html index.htm;
    }

    location ~ \.php$ {
        root           /usr/local/www/nginx;
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
}
```

VII: Create database and user

```bash
mysql -u root -p
CREATE DATABASE nsa501;
CREATE USER 'backend'@'localhost' IDENTIFIED BY 'Bit8Q6a6G;
GRANT ALL PRIVILEGES ON nsa501.* TO 'backend'@'localhost;
FLUSH PRIVILEGES;
EXIT;
```

VIII: Copy database with the nsa501.sql file

```bash
mysql -u backend -p nsa501 < /nsa501.sql
```

IX: Copy the data.php script file

```bash
cp data.php /usr/local/www/nginx
```

Well done, you succesfully configured the web server!

## VM3 & VM4

Both client machines can be installed with the system of your choice and with a GUI, the network configuration being automatically recovered by the DHCP