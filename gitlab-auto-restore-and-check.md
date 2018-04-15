---
title: compare repo last_activity_at between two gitlab instances
date: 2018-04-13 11:13
categories: gitlab
tags: 
- gitlab
- python-gitlab
- gitlab-auto-restore
---


# gitlab-auto-restore

``` shell

# Options: 清理之前备份还原时产生的临时文件
# [ -e /data/var/opt/gitlab/git-data/repositories.old.* ] && rm -rf /data/var/opt/gitlab/git-data/repositories.old.*
find /data/var/opt/gitlab/git-data -maxdepth 1 -name 'repositories.old.*' -type d -print0 |xargs -0 -I {} rm -rf "{}"

# 复制文件到备份目录
cp  ${BAK_FILE_PATH} /data/var/opt/gitlab/backups/

# 执行恢复
BACKUP_INFO=$(ls  /data/var/opt/gitlab/backups/|head -n1|sed 's/_gitlab_backup.tar//')

sudo chown -R git:git /data/var/opt/gitlab/backups
sudo gitlab-ctl stop unicorn
sudo gitlab-ctl stop sidekiq
sudo gitlab-rake gitlab:backup:restore  force=yes BACKUP=${BACKUP_INFO} --trace
sudo gitlab-ctl start
sudo gitlab-rake gitlab:check SANITIZE=true

```



# compare repo last_activity_at between two gitlab instances

``` shell

# install pip python-gitlab
apt-get install python-pip
pip install --upgrade  python-gitlab
```


``` python

#!/usr/bin/env python
import sys
from datetime import datetime,timedelta
import gitlab
gl = gitlab.Gitlab('http://gitlab.server.url', '********************', api_version=4)
gltest = gitlab.Gitlab('http://gitlabtest.server.url', '********************', api_version=4)
gl.auth()
now_utc=datetime.utcnow()
print 'now_utc %s'%now_utc
now_utc_ts=float(now_utc.strftime("%s"))
print 'now_utc_ts %d'%now_utc_ts
bak_utc=now_utc - timedelta(hours=8)
bak_utc_ts=float(bak_utc.strftime("%s"))
def str_to_ts(s):
    dt_utc=datetime.strptime(s, '%Y-%m-%dT%H:%M:%S.%fZ')
    return float(dt_utc.strftime("%s"))

projects = gl.projects.list(all=True)
for p in projects:
    ptest=None
    try:
        if p.ssh_url_to_repo in ( 'git@gitlab.server.url:ligyj/xxx-web.git', 'git@gitlab.server.url:ymj/xxx-count-server.git') :
            continue
        ptest=gltest.projects.get(p.id)
    except Exception as e:
        print '[Exception] except :p.id is  %d, %s\t%s'%(p.id, p.ssh_url_to_repo, p.last_activity_at)
        print '[Exception] %s'%str(e)
    last_act_timestamp = str_to_ts(p.last_activity_at)
    print p.last_activity_at
    print last_act_timestamp
    if last_act_timestamp > bak_utc_ts:
        print "[INFO] UPDATE code after backup: %s %s > %s" % (p.ssh_url_to_repo,p.last_activity_at,bak_utc)
        if ptest == None :
           print "[INFO] NEW repo %s %s > %s"%(p.ssh_url_to_repo,p.last_activity_at,bak_utc)
    else:
        if ptest == None :
            print "[ERROR] LOST repo %s %s"%(p.ssh_url_to_repo,p.last_activity_at)
            sys.exit(9)
        elif (p.last_activity_at != ptest.last_activity_at):
            print "[ERROR] LOST code: %s p(%s) \t=>\t ptest(%s)"%(p.ssh_url_to_repo,p.last_activity_at, ptest.last_activity_at)
            sys.exit(9)
        else:
            pass
```
