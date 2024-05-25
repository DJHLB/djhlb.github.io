---
title: How to Install Latest MySQL5.7 on CentOS7
date: 2021-07-05 09:02:44
tags:
---
Checking if installed MariaDB by default.

```shell
rpm -qa | grep -i mariadb
#Uninstalling
rpm -e --nodeps mariadb-libs-*.x86_64
```

You can visit the MySQL community Yum Repository which provides packages for MySQL.

```shell
wget https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
```

or:

```shell
curl -sSLO https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
```

Checking the MD5 value.

```shell
md5sum mysql57-community-release-el7-11.noarch.rpm
```

Installing the package.

```shell
rpm -ivh mysql57-community-release-el7-11.noarch.rpm
```

or:

```shell
yum localinstall mysql57-community-release-el7-11.noarch.rpm
```

Importing the GPG KEY.

```shell
rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
```

Verifying that MySQL Yum has been successfully added.

```shell
yum repolist enabled | grep "mysql.*-community.*"
```

Installing MySQL.

```shell
yum install mysql-community-server -y
```

Starting the service of MySQL.

```shell
systemctl start mysqld
#Checking the status of MySQL
systemctl status mysqld
```

 Copying the temporary password.

```shell
grep 'temporary password' /var/log/mysqld.log
```

Updating a new password for MySQL root.

```shell
mysql_secure_installation
```
