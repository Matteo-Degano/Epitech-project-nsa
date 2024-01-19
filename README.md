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

I: ``` /etc/dhcpd.conf ```
data : ``` subnet 192.168.42.0 netmask 255.255.255.192 ```

II : ``` /etc/hostname.em1 ```
data : ``` inet 192.168.42.1 255.255.255.192 ```

III : ``` /etc/hostname.em2 ```
data : ``` net 192.168.42.65 255.255.255.192 ```

IV : ``` /etc/hostname.em3 ```
data : ``` inet 192.168.42.129 255.255.255.192 ```

Start the DHCP service

I: ``` rcctl enable dhcpd ```
II:  ``` tcctl start dhcpd ```

#### - Security

Create the followwing files and fill them with the following data:

I: ``` /etc/pf.conf ```

II: ``` pfctl -f /etc/pf.conf ```

III: ``` pfctl -e ```

####- Internet : 

Go to ```/etc/syscl.conf```
And put in ```net.inet.ip.forwarding=1```

Congratulations, you just have configured the first VM !

## VM2 (Web Server)

### Install

ISO FreeBSD 13.2
<https://download.freebsd.org/ftp/releases/ISO-IMAGES/13.2/FreeBSD-13.2-RELEASE-amd64-bootonly.iso>

### Config

#### - Nginx

I: ```pkg install nginx```

II: ```service nginx enable```

III: ```service nginx start```

#### - Php 8.1

I: ```pkg install php81```

II: ```cp -v /usr/local/etc/php.ini-production /usr/local/etc/php.ini```

III: ```pkg install vim php81-xml mod_php81 php81-zip php81-mbstring php81-zlib php81-curl php81-mysqli php81-gd php81-gd```

IV: ```service php-fpm enable```
    ``` service php-fpm start```

#### - MySQL

I: ```pkg install mysql180-server```

II: ```service mysql-server onestart```

III: ```mysql_secure_installation```

IV: ```service mysql-server enable```
```service mysql-server start```

V: Update /etc/rc.conf with those datas : 
```defaultrouter="192.168.42.65" (ip de la gateway)
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
}```

VII: Create database and user
```mysql -u root -p```
```CREATE DATABASE nsa501```;
```CREATE USER 'backend'@'localhost' IDENTIFIED BY 'Bit8Q6a6G';```
```GRANT ALL PRIVILEGES ON nsa501.* TO 'backend'@'localhost';```
```FLUSH PRIVILEGES;```
```EXIT```

VIII: Copy database with the nsa501.sql file
```mysql -u backend -p nsa501 < /nsa501.sql```

IX: Copy the data.php script file 
```cp data.php /usr/local/www/nginx```

Well done, you succesfully configured the web server !