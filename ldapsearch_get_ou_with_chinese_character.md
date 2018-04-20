---
title: ldapsearch get ou with Chinese character
date: 2018-04-11 10:01
categories: backup
tags: 
- ldapsearch
- Base64
- decode_base64
- perl
---

``` shell
ldapsearch  -LLL -x -h ad.ddd.com.cn:3268 -b "OU=公司,OU=集团,DC=ddd,DC=com,DC=cn" "(objectClass=organizationalUnit)"  -D "CN=accountname,DC=ddd,DC=com,DC=cn" "distinguishedName"  -W >distinguishedName.txt
cat distinguishedName.txt | awk 'BEGIN { RS = "" ; FS = ":" } { gsub("\n",""); ds=substr($0,index($0,"distinguishedName::")); gsub(" ","",ds);  print ds; }' |awk -F'::' '{ print $2;}' >aa.txt 
perl -MMIME::Base64 -ne 'print decode_base64($_); print "\n\n\n"' < aa.txt   | iconv -f utf8 >d2.txt 
cat d2.txt |grep -v ','|grep -v '撤销'|egrep -e'TC|CC|研发|开发|测试|产品|培训|T|UED|技术'|sort -u
```
