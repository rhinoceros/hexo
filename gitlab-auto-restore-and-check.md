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
''' diff projects on two gitlab server '''
import sys
from datetime import datetime, timedelta
import logging
import gitlab

def str_to_ts(str_date):
    ''' convert string(%Y-%m-%dT%H:%M:%S.%fZ) to float(ts) '''
    dt_utc = datetime.strptime(str_date, '%Y-%m-%dT%H:%M:%S.%fZ')
    return float(dt_utc.strftime("%s"))

def main():
    ''' main function '''
    now_utc = datetime.utcnow()
    now_utc_ts = float(now_utc.strftime("%s"))
    logging.info('now_utc: {}, ts: {}'.format(now_utc, now_utc_ts))

    bak_utc = now_utc - timedelta(hours=10)
    bak_utc_ts = float(bak_utc.strftime("%s"))

    lostproj_repos = []
    lostcode_repos = []

    projects = GL_PROD.projects.list(all=True)
    gltest_projects = GL_TEST.projects.list(all=True)
    gltest_projects_dict = dict((x.id, x) for x in gltest_projects)

    for proj in projects:
        if proj.last_activity_at == proj.created_at:
            continue

        ptest = gltest_projects_dict.get(proj.id, None)
        last_act_timestamp = str_to_ts(proj.last_activity_at)
        if last_act_timestamp < bak_utc_ts:
            if ptest == None:
                lostproj_repos.append(proj)
                continue
            if proj.last_activity_at != ptest.last_activity_at:
                lostcode_repos.append(proj)

    for proj in lostproj_repos:
        logging.error("LOST repo: {}    {}"
                     .format(proj.ssh_url_to_repo, proj.last_activity_at))
    for proj in lostcode_repos:
        logging.error("LOST code: {} proj: {}    ptest: {}"
                     .format(proj.ssh_url_to_repo, proj.last_activity_at,
                     ptest.last_activity_at))
    if len(lostproj_repos) > 0 or len(lostcode_repos) > 0:
        sys.exit(9)


PRIVATE_TOKEN = "********************"
GL_PROD = gitlab.Gitlab('http://prod.server.url', PRIVATE_TOKEN)
GL_TEST = gitlab.Gitlab('http://test.server.url', PRIVATE_TOKEN)


if __name__ == "__main__":
    main()
```
