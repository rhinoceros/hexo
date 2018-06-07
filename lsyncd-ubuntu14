---
title: lsyncd ubuntu14
date: 2018-06-07 20:01
categories: lsyncd
tags: 
- lsyncd
---


``` shell

#!/bin/bash
set -eu

APPNAME="jira"

#apt-get install -y lsyncd
#sysctl fs.inotify.max_user_watches=819200
mkdir /etc/lsyncd
mkdir /var/log/lsyncd
touch /etc/lsyncd/lsyncd-excludes.txt
echo '****************' > /etc/lsyncd/rsync.172.18.x.x.pwd
chmod 600 /etc/lsyncd/rsync.172.18.x.x.pwd


cat >/etc/lsyncd/lsyncd.conf.lua <<EOF
settings {
        logfile = "/var/log/lsyncd/lsyncd.log",
        statusFile = "/var/log/lsyncd/lsyncd-status.log",
        insist = true
}
sync {
    default.rsync,
    source = "/data/atlassian/application-data/jira",
    target = "mirror_user1@172.18.x.x::mirror/${APPNAME}",
    excludeFrom = "/etc/lsyncd/lsyncd-excludes.txt",
    delete = "running",
    delay = 30,
    rsync  = {
        binary = "/usr/bin/rsync",
        archive = true,
        compress = true,
        verbose   = true,
        password_file = "/etc/lsyncd/rsync.172.18.x.x.pwd",
        _extra    = {"--bwlimit=500000"}
        }
}
EOF


cat >/etc/lsyncd/lsyncd-excludes.txt <<EOF
caches/
log/
logs/
tmp/
EOF


cat >/etc/logrotate.d/lsyncd <<EOF
/var/log/lsyncd/*log {
    missingok
    notifempty
    sharedscripts
    postrotate
    if [ -f /var/lock/lsyncd ]; then
      /usr/sbin/service lsyncd restart > /dev/null 2>/dev/null || true
    fi
    endscript
} 
EOF

```



# https://www.stephenrlang.com/2015/12/how-to-install-and-configure-lsyncd/


```
Ubuntu 14.04 and 16.04 – How To Install Lsyncd

Install Lsyncd via apt. Please note, this will automatically setup
– Lsyncd 2.1.5
– /etc/init.d/lsyncd

But it will not setup:
– /etc/logrotate.d/lsyncd
– /etc/lsyncd/lsyncd.conf.lua

Install it by running

apt-get update
apt-get install lsyncd
Now setup the lsyncd configuration by:

mkdir /etc/lsyncd
vim /etc/lsyncd/lsyncd.conf.lua
 
settings {
   logfile = "/var/log/lsyncd/lsyncd.log",
   statusFile = "/var/log/lsyncd/lsyncd-status.log",
   statusInterval = 20
}
servers = {
 "x.x.x.x",
 "x.x.x.x",
 "x.x.x.x"
}
 
for _, server in ipairs(servers) do
sync {
    default.rsyncssh,
    source="/var/www/",
    host=server,
    targetdir="/var/www/",
    excludeFrom="/etc/lsyncd/lsyncd-excludes.txt",
    rsync = {
        compress = true,
        archive = true,
        verbose = true,
        rsh = "/usr/bin/ssh -p 22 -o StrictHostKeyChecking=no"
    }
}
end
Create the place holder for lsyncd-excludes.txt and logs directory

touch /etc/lsyncd/lsyncd-excludes.txt
mkdir /var/log/lsyncd
Setup log rotate script

vim /etc/logrotate.d/lsyncd

/var/log/lsyncd/*log {
    missingok
    notifempty
    sharedscripts
    postrotate
    if [ -f /var/lock/lsyncd ]; then
      /usr/sbin/service lsyncd restart > /dev/null 2>/dev/null || true
    fi
    endscript
}
Finally, start the service

service lsyncd start
Note: On Ubuntu 16.04, if ‘service lsyncd start’ does not start daemon, then manually start lsyncd, which will allow systemd to work with it from here on out by running:

/usr/bin/lsyncd /etc/lsyncd/lsyncd.conf.lua
service lsyncd stop
service lsyncd start
```


