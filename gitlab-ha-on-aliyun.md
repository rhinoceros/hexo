---
title: gitlab-ha on aliyun
date: 2019-03-12 15:32
categories: gitlab
tags: 
- gitlab
- ha
- aliyun
---


### 准备条件
```
    1 个SLB： 转发http80和tcp22
    2台ECS， 8c16g
    1个Redis: gitlab-ha-xxxxx.redis.rds.aliyuncs.com
    1个postgres sql database（版本10.1）
            （CREATE EXTENSION pg_trgm;）
             gitlab-ha-xxxxx.pg.rds.aliyuncs.com
    1个NAS : gitlab-ha-xxxxx.nas.aliyuncs.com
```

### ALL NODES 配置


```
apt-get install nfs-common
 
# https://help.aliyun.com/document_detail/90492.html?spm=a2c4g.11186623.6.562.3b645eb2EcWCj6
# Linux NFS 客户端对于同时发起的NFS请求数量进行了控制，若该参数配置较小，会降低 IO 性能。
# 默认编译的内核中该参数最大值为256。您可以使用root用户执行以下命令来提高该参数的值，取得较好的性能。
echo "options sunrpc tcp_slot_table_entries=128" >> /etc/modprobe.d/sunrpc.conf
echo "options sunrpc tcp_max_slot_table_entries=128" >>  /etc/modprobe.d/sunrpc.conf
sysctl -w sunrpc.tcp_slot_table_entries=128
 


mkdir /gitlab-nfs
# vim /etc/fstab
# 	    gitlab-ha-xxxxx.nas.aliyuncs.com:/gitlab-nfs /gitlab-nfs nfs4 defaults,soft,rsize=1048576,wsize=1048576,noatime,nofail,lookupcache=positive,timeo=600,retrans=2,noresvport 0 2


mount -a
mkdir -p /gitlab-nfs/gitlab-data/git-data
mkdir -p /gitlab-nfs/gitlab-data/home
mkdir -p /gitlab-nfs/gitlab-data/uploads
mkdir -p /gitlab-nfs/gitlab-data/shared
mkdir -p /gitlab-nfs/gitlab-data/builds

# vim /etc/gitlab/gitlab.rb
# 			 external_url 'http://gitlab-ha.rd.company.com'
# 
# 			 # Prevent GitLab from starting if NFS data mounts are not available
# 			 high_availability['mountpoint'] = '/gitlab-nfs'
# 
# 			 # Disable components that will not be on the GitLab application server
# 			 roles ['application_role']
# 			 nginx['enable'] = true
# 			 postgresql['enable'] = false
# 			 redis['enable'] = false
# 
# 			 gitlab_rails['time_zone'] = 'Asia/Shanghai'
# 
# 			 # PostgreSQL connection details
# 			 gitlab_rails['db_adapter'] = 'postgresql'
# 			 gitlab_rails['db_encoding'] = 'unicode'
# 
# 			 gitlab_rails['db_host'] = 'gitlab-ha-xxxxx.pg.rds.aliyuncs.com'
# 
# 			 gitlab_rails['db_port'] = 3433
# 
# 			 gitlab_rails['db_username'] = "gitlab_ha"
# 			 gitlab_rails['db_password'] = '*********'
# 			 gitlab_rails['db_database'] = "gitlab_ha"
# 
# 
# 			 # Redis connection details
# 			 gitlab_rails['redis_port'] = '6379'
# 			 gitlab_rails['redis_host'] = "gitlab-ha-xxxxx.redis.rds.aliyuncs.com"
# 			 gitlab_rails['redis_password'] = "*************"
# 
# 			 # Ensure UIDs and GIDs match between servers for permissions via NFS
# 			 user['uid'] = 9000
# 			 user['gid'] = 9000
# 			 web_server['uid'] = 9001
# 			 web_server['gid'] = 9001
# 			 registry['uid'] = 9002
# 			 registry['gid'] = 9002

```



### First-GitLab-application-server 配置
```
sudo apt-get update
sudo apt-get install -y curl openssh-server ca-certificates
dpkg -i gitlab-ce_11.8.0-ce.0_amd64.deb
sudo gitlab-rake gitlab:setup
sudo gitlab-ctl reconfigure
```

### Additional-GitLab-application-servers 配置
```
sudo apt-get update
sudo apt-get install -y curl openssh-server ca-certificates
dpkg -i gitlab-ce_11.8.0-ce.0_amd64.deb
 
 
# 把First-GitLab-application-server gitlab-secrets.json 和 ssh 到当前节点，以下命令未测试，只表明意图
# scp xx@First-GitLab-application-server:/etc/gitlab/gitlab-secrets.json /etc/gitlab/
# cp -rf /etc/ssh/ /etc/ssh-bak/
# scp -r xx@First-GitLab-application-server:/etc/ssh/ /etc/ssh/

service ssh restart
touch /etc/gitlab/skip-auto-reconfigure
sudo gitlab-ctl reconfigure

```
