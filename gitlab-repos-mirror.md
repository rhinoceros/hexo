---
title: mirror repository between two gitlab instances
date: 2018-04-18 18:47
categories: git
tags: 
- gitlab
- git
- mirror
---



you can trigger this script when 'git@gitlab-source.server.url:group/project.git' have push events

   

``` shell
#!/bin/bash
set -e
if [ -e project-local ]; then
    cd project-local
    git fetch -f -p origin
else
    git clone --mirror git@gitlab-source.server.url:group/project.git project-local
    
fi

cd ${WORKSPACE}/project-local
git push -f --mirror http://USER:PASSWORD@gitlab-dest.server.url/group/project.git

```
