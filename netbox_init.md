---
title: netbox_init
date: 2021-04-13 10:21:03
categories:  [netbox]
tags: [netbox]
---


<!--more-->


# netbox_init
https://netbox.readthedocs.io/en/stable/installation/3-netbox/



## 安装 postgresql

sudo apt update
sudo apt install -y postgresql libpq-dev


sudo systemctl start postgresql
sudo systemctl enable postgresql
sudo systemctl status postgresql


状态
![](https://noback.upyun.com/2021-04-13-10-19-46.png!)


> init postgresql databases

```bash
$ sudo -u postgres psql
psql (12.5 (Ubuntu 12.5-0ubuntu0.20.04.1))
Type "help" for help.

postgres=# CREATE DATABASE netbox;
CREATE DATABASE
postgres=# CREATE USER netbox WITH PASSWORD 'J5brHrAXFLQSif0K';
CREATE ROLE
postgres=# GRANT ALL PRIVILEGES ON DATABASE netbox TO netbox;
GRANT
postgres=# \q
```

> Verify Databases connect

```bash
ubuntu@netbox-5:~$ psql --username netbox --password --host localhost netbox
Password for user netbox:
psql (10.16 (Ubuntu 10.16-0ubuntu0.18.04.1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
Type "help" for help.

netbox=> \conninfo
You are connected to database "netbox" as user "netbox" on host "localhost" at port "5432".
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
netbox=> \q
```

## 安装 redis

```bash
sudo apt install -y redis-server
```

> Verify Databases connect

```bash
$ redis-cli ping
PONG
```

## 安装netbox

> 安装python3环境

```bash
sudo apt install -y python3 python3-pip python3-venv python3-dev build-essential libxml2-dev libxslt1-dev libffi-dev libpq-dev libssl-dev zlib1g-dev
# upgrade pip tool
sudo pip3 install --upgrade pip
```

> 安装netbox

```bash
sudo wget https://github.com/netbox-community/netbox/archive/refs/tags/v2.10.9.tar.gz
sudo tar -xvf v2.10.9.tar.gz -C /opt/
sudo ln -s /opt/netbox-2.10.9/ /opt/netbox

>> ubuntu@netbox-5:~$ ls -l /opt | grep netbox
lrwxrwxrwx 1 root root   19 Apr 13 02:29 netbox -> /opt/netbox-2.10.9/
drwxrwxr-x 7 root root 4096 Apr 12 17:28 netbox-2.10.9
```

> create user

```bash
sudo adduser --system --group netbox
sudo chown --recursive netbox /opt/netbox/netbox/media/
```

> configuration

```bash
cd /opt/netbox/netbox/netbox/
sudo cp configuration.example.py configuration.py
# change configuration.py  ALLOWED_HOSTS = ['*']
# change configuration.py databases connect 
DATABASE = {
    'NAME': 'netbox',               # Database name
    'USER': 'netbox',               # PostgreSQL username
    'PASSWORD': 'J5brHrAXFLQSif0K', # PostgreSQL password
    'HOST': 'localhost',            # Database server
    'PORT': '',                     # Database port (leave blank for default)
    'CONN_MAX_AGE': 300,            # Max database connection age (seconds)
}
# change configuration.py redis connect

REDIS = {
    'tasks': {
        'HOST': 'localhost',      # Redis server
        'PORT': 6379,             # Redis port
        'PASSWORD': '',           # Redis password (optional)
        'DATABASE': 0,            # Database ID
        'SSL': False,             # Use SSL (optional)
    },
    'caching': {
        'HOST': 'localhost',
        'PORT': 6379,
        'PASSWORD': '',
        'DATABASE': 1,            # Unique ID for second database
        'SSL': False,
    }
}
```