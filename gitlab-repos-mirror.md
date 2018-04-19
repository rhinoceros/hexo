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
