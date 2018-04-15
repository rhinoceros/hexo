---
title: bak restore with alicloud-duplicity
date: 2018-04-11 10:01
categories: backup
tags: 
- alicloud-duplicity
- backup
- restore
---

# 参考链接
``` shell
echo from: http://duplicity.nongnu.org/index.html
echo from: https://yq.aliyun.com/articles/197921
echo from: https://www.digitalocean.com/community/tutorials/how-to-use-duplicity-with-gpg-to-securely-automate-backups-on-ubuntu#create-ssh-and-gpg-keys
```

# 备份(on bak-server)：
```shell
#INSTALL alicloud-duplicity (Ubuntu 源码)
sudo apt install librsync-dev python-pip python-dev -y
sudo pip install oss2 fasteners configparser setuptools
 
git clone https://github.com/aliyun/alicloud-duplicity.git
cd alicloud-duplicity
sudo python setup.py install
 
#gpg gen-key
apt-get install gpg-agent
#        mkdir /tmp/.gpg-agent-$USER
#        chmod 700 /tmp/.gpg-agent-$USER
#        gpg-agent --daemon --write-env-file /tmp/.gpg-agent-$USER/env
#        . /tmp/.gpg-agent-$USER/env
# export GPG_AGENT_INFO
# echo $GPG_AGENT_INFO
 
gpg --gen-key
gpg --list-keys

#(option) 配置文件为~/.alicloud.cfg, 也可在执行脚本时配置变量
[oss]
endpoint = http://oss-cn-hangzhou.aliyuncs.com
access_key_id = Jwd12S**********ZBs
access_key_secret = ir8Qt4h0kwx********u1SpmxM5QE
 

#执行备份
#BACKUP ON BAK_SERVER
export ALICLOUD_OSS_ENDPOINT=http://oss-cn-hangzhou.aliyuncs.com
export ALICLOUD_ACCESS_KEY_ID=Jwd12S**********ZBs
export ALICLOUD_ACCESS_KEY_SECRET=ir8Qt4h0kwx********u1SpmxM5QE
export PASSPHRASE="***"
PASSPHRASE="passphrase_for_GPG" alicloud-duplicity  full --encrypt-key 05AB3DF5 /source oss://bucket-name/keyfolder/
```


# GPG-KEY分发 (从 bak-server 分发到 restore-server)
``` shell 
#ON bak-server
gpg --list-keys
gpg --armor --export bak-key-name > bak-key-name.asc
gpg --armor --export-secret-keys bak-key-name > bak-key-name.secret.asc
cat bak-key-name.secret.asc |nc -l bak-server_IP 12345

#ON restore-server
nc bak-server_IP 12345 > bak-key-name.secret.asc
gpg --import bak-key-name.secret.asc
```


# 恢复(on restore-server)
``` shell
#INSTALL alicloud-duplicity (Ubuntu 源码)
sudo apt install librsync-dev python-pip python-dev -y
sudo pip install oss2 fasteners configparser setuptools
 
git clone https://github.com/aliyun/alicloud-duplicity.git
cd alicloud-duplicity
sudo python setup.py install

#执行恢复 

export ALICLOUD_OSS_ENDPOINT=http://oss-cn-hangzhou.aliyuncs.com
export ALICLOUD_ACCESS_KEY_ID=Jwd12S**********ZBs
export ALICLOUD_ACCESS_KEY_SECRET=ir8Qt4h0kwx********u1SpmxM5QE
export PASSPHRASE="***"
#RESTORE ON RESTORE_SERVER
PASSPHRASE="passphrase_for_GPG" alicloud-duplicity  restore --encrypt-key 05AB3DF5  oss://bucket-name/keyfolder/ /source-restore
```


