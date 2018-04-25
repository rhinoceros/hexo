---
title: mirror repositories between two gitlab instances
date: 2018-04-19 11:32
categories: git
tags: 
- gitlab
- git
- mirror
---

### prerequisites
```
  two gitlab:  gitlab-source.server.url  gitlab-dest.server.url
               group name :  both gitlab instances have op group
               permission :  create user and set the user as the  master of op group.
               gitlab (api version=4)
               
  jenkins with gitlab plugin, job-dsl plugin
  
  python2.7 with python-gilab
  
```
  

### create project on gitlab-dest.server.url

``` python
#!/usr/bin/env python
import sys
import gitlab

gl = gitlab.Gitlab('http://gitlab-source.server.url', '*****PrivateToken*****', api_version=4)
gl_prod = gitlab.Gitlab('http://gitlab-dest.server.url', '*****PrivateToken-on-dest*****', api_version=4)


group = gl.groups.list(search="op")[0]
print group.name
print group.id

group_prod = gl_prod.groups.list(search="op")[0]
print group_prod.name
print group_prod.id

projects = group.projects.list(all=True)
projects_prod = group_prod.projects.list(all=True)

for project in projects:
    for project_prod in projects_prod:
        if project_prod.name != project.name:
            print project.name
            gl_prod.projects.create({'name': project.name , 'namespace_id': group_prod.id})

``` 


### You can create an jenkins job with a step contains below script

``` shell
#!/bin/bash
set -e
if [ -e project-script ]; then
    cd project-script
    git fetch -f -p origin
else
    git clone --mirror git@gitlab-source.server.url:op/project-script.git project-script
    
fi

cd ${WORKSPACE}/project-script
git push -f --mirror http://USER:PASSWORD@gitlab-dest.server.url/op/project-script.git

```


### create jenkins job with DSL
``` groovy
PROJECT_LISTS=['tools-op',
'project-script',
]

PROJECT_LISTS.each{item->
    job_name="mirror-to-gitlab-dest__op__${item}"
    println "[INFO] ${item} --> ${job_name}"
    shell_text='''#!/bin/bash
set -e
if [ -e project-script ]; then
    cd project-script
    git fetch -f -p origin
else
    git clone --mirror git@gitlab-source.server.url:op/project-script.git project-script
    
fi

cd ${WORKSPACE}/project-script
git push -f --mirror http://USER:PASSWORD@gitlab-dest.server.url/op/project-script.git
'''.replace('project-script',item.toLowerCase())

job(job_name)  {
  label("master")  
    triggers {
        gitlabPush {
            buildOnMergeRequestEvents(false)
            buildOnPushEvents(true)
            enableCiSkip(true)
            setBuildDescription(true)
            rebuildOpenMergeRequest('never')
            targetBranchRegex('')
        }
    }
    configure {
        it / triggers / 'com.dabsquared.gitlabjenkins.GitLabPushTrigger' << secretToken('JENKINS_JOB_GitLabPushTrigger_SECRETTOKEN')
    }

    steps {
      shell(shell_text)
    }
  }
}
```



### add hooks to gitlab-project
when 'git@gitlab-source.server.url:op/project-script.git' have push events

```
import gitlab

group_name='op'
project_name_list=['tools-op',
'project-script',
]


gl = gitlab.Gitlab('http://gitlab-source.server.url', '*****PrivateToken*****', api_version=4)

for project_name in project_name_list:
    print group_name+'/'+project_name
    proj = gl.projects.get(group_name+'/'+project_name)
    print "%s"%proj.id
    proj_ci_url='http://jenkins.server.url/project/mirror-to-gitlab-dest__op__'+project_name
    print proj_ci_url
    hook = gl.project_hooks.create({'url': proj_ci_url,'issues_events': 0 ,'merge_requests_events':0,  'tag_push_events': 1, 'push_events': 1,'enable_ssl_verification': 0, 'token':'JENKINS_JOB_GitLabPushTrigger_SECRETTOKEN'}, project_id=proj.id)
    hooks = proj.hooks.list()
    for hook in hooks:
        if hook.url==proj_ci_url:
            print proj_ci_url
```


### Another way!

```
python 2.7
apt-get install git-all
apt-get install python-pip
pip install --upgrade python-gitlab
pip install gitpython
ssh-keygen
Add id_rsa.pub to gitlab user's ssh-keys

```
```
#!/usr/bin/env python
''' run gitlab mirror job '''
import os.path as osp
from datetime import datetime
import time
import gitlab
from git import Repo
import shutil
import logging

def get_run_config():
    ''' get run config '''
    run_config = "enable"
    with open('run.config', 'r') as f_obj:
        run_config = f_obj.read().strip()
    f_obj.close()
    return run_config

def set_run_config(run_config):
    ''' set run config '''
    with open('run.config', 'w+') as f_obj:
        f_obj.write(run_config)
    f_obj.close()

def read_last_date():
    ''' read last run date '''
    last_date = ""
    with open('run.lastdate', 'r') as f_obj:
        last_date = f_obj.read().strip()
    f_obj.close()
    return last_date

def write_last_date(last_date):
    ''' write last run date '''
    with open('run.lastdate', 'w+') as f_obj:
        f_obj.write(last_date)
    f_obj.close()

def remove_dir(dir_path):
    ''' remove dir '''
    logging.info("begin remove_dir : " + dir_path)
    shutil.rmtree(dir_path, ignore_errors=True)
    while osp.exists(dir_path):
        pass
    print "[INFO] done  remove_dir : " + dir_path

def mirror_repo(source_url, local_path, dest_url):
    ''' mirror repo '''
    if osp.exists(local_path):
        repo = Repo(local_path)
        if repo.remotes.origin.url == source_url:
            repo.remotes.origin.fetch(prune=True, force=True)
        else:
            remove_dir(local_path)
            repo = Repo.clone_from(source_url, local_path, mirror=True)
    else:
        repo = Repo.clone_from(source_url, local_path, mirror=True)

    if any(r.name == "mirror" for r in repo.remotes):
        logging.info("Exist remote mirror")
        if repo.remotes.mirror.url != dest_url:
            logging.info("Set mirror url: " + dest_url)
            repo.remotes.mirror.url = dest_url
    else:
        repo.create_remote("mirror", dest_url)

    repo.remotes.mirror.push(mirror=True)

def str_to_ts(str_date):
    ''' convert string(%Y-%m-%dT%H:%M:%S.%fZ) to float(ts) '''
    dt_utc = datetime.strptime(str_date, '%Y-%m-%dT%H:%M:%S.%fZ')
    return float(dt_utc.strftime("%s"))

def run_mirror(last_date):
    ''' run mirror '''
    logging.info("run mirror: {}".format(last_date))
    lastdate_utc_ts = str_to_ts(last_date)

    projects = []
    i = 0
    while True:
        i = i+1
        logging.info("page id {}".format(i))
        projects_i = GL_PROD.projects.list(order_by="last_activity_at",
                                           page=i, per_page=40)
        if len(projects_i) == 0:
            break
        projects.extend(projects_i)
        if len(projects) == 0:
            break
        if str_to_ts(projects[-1].last_activity_at) < lastdate_utc_ts:
            break

    for p_src in projects:
        logging.info("p_src.id is {}: ".format(p_src.id))
        p_dest = GLTEST_PROJECTS_DICT.get(p_src.id, None)
        if p_dest == None:
            logging.info("Not found {} {} in GLTEST"
                         .format(p_src.id, p_src.ssh_url_to_repo))
            continue

        last_activity_at_ts = str_to_ts(p_src.last_activity_at)
        if last_activity_at_ts > lastdate_utc_ts:
            logging.info("Mirror Repo : {} p: {} "
                         .format(p_src.ssh_url_to_repo,
                         p_src.last_activity_at))
            source_url = p_src.ssh_url_to_repo
            local_path = osp.join(MIRROR_LOCAL_ROOT, p_src.path_with_namespace)
            dest_url = p_dest.ssh_url_to_repo
            mirror_repo(source_url, local_path, dest_url)

    return True


def main():
    ''' main func '''
    while True:
        logging.info("This run once a 5 minutes.")
        time.sleep(10)
        run_config = get_run_config()
        if run_config == "stop":
            break

        if run_config == "enable":
            last_date = read_last_date()
            now_date = datetime.utcnow().strftime('%Y-%m-%dT%H:%M:%S.%fZ')
            if last_date == "":
                last_date = now_date

            if run_mirror(last_date):
                write_last_date(now_date)


PRIVATE_TOKEN = "******************"
GL_PROD = gitlab.Gitlab('http://gitlab.prod.server.url', PRIVATE_TOKEN)
GL_TEST = gitlab.Gitlab('http://gitlab.test.server.url', PRIVATE_TOKEN)
MIRROR_LOCAL_ROOT = '/data/gitlab-mirror'

print datetime.utcnow().strftime('%Y-%m-%dT%H:%M:%S.%fZ')
GLTEST_PROJECTS = GL_TEST.projects.list(all=True)
GLTEST_PROJECTS_DICT = dict((x.id, x) for x in GLTEST_PROJECTS)
print datetime.utcnow().strftime('%Y-%m-%dT%H:%M:%S.%fZ')

if __name__ == "__main__":
    logging.basicConfig(level=logging.INFO)
    main()
```
