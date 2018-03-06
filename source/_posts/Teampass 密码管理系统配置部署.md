title: Teampass 密码管理系统配置部署
tags:
  - TEAMPASS
categories:
  - OPEN SOURCE
author: NPC
date: 2017-11-02 06:49:00
---
![TEAMPASS](/images/pasted-0.png)
TeamPass是一套密码管理系统，用于团队成员之间共享密码来进行协作管理。并为每个用户定义的访问权限，允许以有组织的方式管理您的密码和相关数据。支持开源协议AGPL3。

<!--more-->

### 1.准备LAMP环境
   注意：PHP版本大于5.5
   
    yum info php                    #查看yum仓库PHP版本
    yum install -y httpd php php-fpm mariadb-server openssl      #安装，默认apache已安装


### 2.安装PHP拓展
    php-mcrypt
    php-mbstring
    php-bcmath
    php-iconv
    php-gd
    php-mysqlnd
    php-xml
    php-ldap
    mod_ssl



### 3.修改PHP参数max_execution_time，默认为30
    php.ini  =>  max_execution_time = 60


### 4.配置数据库

	mysql_secure_installation        #初始化mariadb安全配置，设置root密码，删除test库等
    mysql -uroot -p                        #新建teampass所需的库
      >>>create database teampass character set utf8 collate utf8_bin;
      >>>grant all privileges on teampass.* to teampass_admin@localhost identified by 'PASSWORD';


### 5.配置TeamPass站点文件
   https://github.com/nilsteampassnet/TeamPass/tags

	chown -R apache:apache /opt/teampass/
    chmod -R 0777 /opt/teampass/includes/config
    chmod -R 0777 /opt/teampass/includes/avatars
    chmod -R 0777 /opt/teampass/includes/libraries/csrfp/libs
    chmod -R 0777 /opt/teampass/includes/libraries/csrfp/log
    chmod -R 0777 /opt/teampass/includes/libraries/csrfp/js
    chmod -R 0777 /opt/teampass/backups
    chmod -R 0777 /opt/teampass/files
    chmod -R 0777 /opt/teampass/install
    chmod -R 0777 /opt/teampass/upload


### 6.生成站点证书，这里使用Windows CA
    openssl genrsa -out /etc/pki/tls/private/teampass.key                #生成证书key
    openssl req -new -key /etc/pki/tls/private/teampass.key -out  /etc/pki/tls/private/teampass.csr -days 730     #通过key生成csr注册信息
    cat teampass.csr                            #查看生成csr文件，并将内容粘贴到注册CA申请过程
    mv /etc/pki/tls/private/certnew.cer /etc/pki/tls/private/teampass.cer             #下载申请的证书，证书为cer格式，将cer格式转换为pem格式
    openssl x509 -inform der -in /etc/pki/tls/private/teampass.cer -out /etc/pki/tls/private/teampass.pem


### 7.配置Apache

    DocumentRoot "/opt/teampass"
    <Directory "/opt/teampass">
      SetOutputFilter DEFLATE
      Options FollowSymLinks
      AllowOverride All
      Require all granted
      DirectoryIndex index.php index.html index.htm default.php default.html default.htm
    </Directory>
    SSLEngine On
    SSLCertificateFile /opt/teampass/ssl/teampass.pem
    SSLCertificateKeyFile /opt/teampass/ssl/teampass.key
    SSLCipherSuite EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH
    SSLProtocol All -SSLv2 -SSLv3
    SSLHonorCipherOrder On



### 8.设置服务自动启动
    systemctl enable mariadb-server
    systemctl enable php-fpm
    systemctl enable httpd
   

### 9.访问WEB安装
    https://$IP_ADDRESS
