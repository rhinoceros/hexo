---
title: shell demo
date: 2018-05-25 21:01
categories: shell
tags: 
- shell
- semver
---

# shell 练习： 判断变量存在； 正则； 数组；字符串替换； sed 替换文件

# 以下脚本只在osx操作系统上运行过

``` shell
#!/bin/bash
set -eux

# check
[ -z ${new_ver+x} ] && { echo "Failure: new_ver is unset, then exit 9."; exit 9; }
filelist[0]='test-array-supported' || ( echo 'Failure: arrays not supported in this version of bash.' && exit 2; )

#https://github.com/fsaintjacques/semver-tool
SEMVER_REGEX="^(0|[1-9][0-9]*)\\.(0|[1-9][0-9]*)\\.(0|[1-9][0-9]*)(\\-[0-9A-Za-z-]+(\\.[0-9A-Za-z-]+)*)?(\\+[0-9A-Za-z-]+(\\.[0-9A-Za-z-]+)*)?$"

if [[ "$new_ver" =~ $SEMVER_REGEX ]]; then
    echo "INFO: input new version: $new_ver"
else
    echo "Failure: $new_ver does not match the semver scheme 'X.Y.Z(-PRERELEASE)(+BUILD)'."
    exit 9
fi


# 将new_ver中的.过滤掉
# new_ver  1.2.3 
# new_ver1 123
new_ver1="${new_ver//./}"


# 模版文件列表
filelist=(
    'product_VERSION_test2888.plist'
    'product_VERSION_test888.plist'
    'product_VERSION_prod2888.plist'
    'product_VERSION_prod888.plist'
)

for filename in "${filelist[@]}";
do
    echo "[INFO] Process: [ $filename ]"
    #新文件名
    new_filename="${filename/_VERSION_/_${new_ver1}_}"
    #根据模版文件生成新文件，先复制，再将文件中的{{ VERSION }}，替换成新版本号
    cp -frp "${WORKSPACE}/plists/${filename}" "${WORKSPACE}/plists/${new_filename}"
    sed -i '' "s/{{ VERSION }}/${new_ver}/g" "${WORKSPACE}/plists/${new_filename}"
done
```



