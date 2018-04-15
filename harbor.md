---
title: update harbor config
date: 2018-04-11 19:53
categories: harbor
tags: 
- harbor
- update
---



# update

``` shell 
sudo docker-compose -f ./docker-compose.yml -f ./docker-compose.notary.yml down -v

vim harbor.cfg

#prepare
sudo ./prepare --with-notary
#start
sudo docker-compose -f ./docker-compose.yml -f ./docker-compose.notary.yml up -d
```
