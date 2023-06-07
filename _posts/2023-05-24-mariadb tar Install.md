---
layout: post
title: MariaDB tar 수동 설치
date: 2023-05-24 00:00 +0800
last_modified_at: 2023-05-24 00:08:25 +0800
tags: [CentOS 8 Stream, MariaDB 수동 설치, tar 수동 설치]
categoie: [MariaBD]
toc:  true
---
**MariaDB CentOS 8 stream Install**!
{: .message }


# MariaDB를 사용자 계정으로 설치.

Root 계정이 아닌 사용자 계정으로 Mysql을 설치 하려 한다. 이렇듯 초기 계정 생성 외에는 Root계정은 필요가 없다.
 * CentOS 8 Steam
 * mariadb-10.5.15-linux-systemd-x86_64.tar.gz

# User 및 Group 생성
Root 권한으로 필요한 계정을 생성한다. 

```
#User 생성 및 그룹 생성
sudo groupadd mysql
sudo useradd -g mysql mysql
sudo passwd mysql
```

# Mysql 압축 해제 및 심볼 링크 설정

```
su mysql
# /home/mysql 위치에 압축을 해제하고 심볼링크를 등록 한다.
cd ~ 
tar zxvf mariadb-10.5.15-linux-systemd-x86_64.tar.gz
ln -s mariadb-10.5.15-linux-systemd-x86_64 mysql
```

#  my.cnf 작성

```
[mysqld]
datadir=/home/mysql/mydata/mysql
socket=/tmp/mysql.sock
port=3306
tmpdir=/home/mysql/temp
collation-server=utf8_general_ci
init-connect='SET NAMES utf8'
character-set-server=utf8
symbolic-links=0
lower_case_table_names=1

[mysqld_safe]
log-error=/home/mysql/logs/mariadb.log
pid-file=/home/mysql/mariadb.pid
```

# mysql dir 생성
```
mkdir -p /home/mysql/mydata
mkdir -p /home/mysql/temp
mkdir -p /home/mysql/logs
```

# Mariadb 설치

```
scripts/mysql_install_db --defaults-file=/home/mysql/my.cnf
```

# Mariadb Server 실행
```
bin/mysqld --defaults-file=/home/mysql/mysql/my.cnf --datadir=/home/mysql/mydata/mysql
```