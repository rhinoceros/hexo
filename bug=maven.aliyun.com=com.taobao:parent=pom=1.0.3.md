---
title: BUG, maven.aliyun.com, com.taobao:parent:pom:1.0.3.
date: 2018-08-31 00:00
categories: Chanjet
tags: 
- maven
- aliyun
- taobao
- parent
---

add dependency to pom.xml
```
<dependency>
    <groupId>com.aliyun.drc</groupId>
    <artifactId>client</artifactId>
    <version>2.2.1.39-cloud</version>
</dependency>
```


***  then, build it:
***  build errors when not using, maven.aliyun.com


```
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 1.541s
[INFO] Finished at: Tue Aug 28 13:04:36 CST 2018
[INFO] Final Memory: 9M/607M
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal on project my-app: Could not resolve dependencies for project com.mycompany.app:my-app:jar:1.0-SNAPSHOT: Failed to collect dependencies for [com.aliyun.drc:client:jar:2.2.1.39-cloud (compile), junit:junit:jar:3.8.1 (test)]: Failed to read artifact descriptor for com.aliyun.drc:client:jar:2.2.1.39-cloud: Could not find artifact com.taobao:parent:pom:1.0.3 in public-repository (http://172.18.4.142:8081/nexus/content/repositories/central) -> [Help 1]
org.apache.maven.lifecycle.LifecycleExecutionException: Failed to execute goal on project my-app: Could not resolve dependencies for project com.mycompany.app:my-app:jar:1.0-SNAPSHOT: Failed to collect dependencies for [com.aliyun.drc:client:jar:2.2.1.39-cloud (compile), junit:junit:jar:3.8.1 (test)]
        at org.apache.maven.lifecycle.internal.LifecycleDependencyResolver.getDependencies(LifecycleDependencyResolver.java:2
        
```

***  Cause: 在maven官方，仓库比在阿里云代理过来的central多了个parent

------------------
http://repo1.maven.org/maven2/com/aliyun/drc/client/2.2.1.39-cloud/client-2.2.1.39-cloud.pom
```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
<modelVersion>4.0.0</modelVersion>
 <!--  delete for public maven deploy  -->
<parent>
<groupId>com.taobao</groupId>
<artifactId>parent</artifactId>
<version>1.0.3</version>
</parent>
<name>client</name>
<url>https://www.aliyun.com/product/dts</url>
<description>
The core java client for accessing Data Transmission Service
</description>
<packaging>jar</packaging>
<groupId>com.aliyun.drc</groupId>
<artifactId>client</artifactId>
<version>2.2.1.39-cloud</version>
```
---------------------------
http://maven.aliyun.com/mvn/search

http://archiva-maven-storage-prod.oss-cn-beijing.aliyuncs.com/repository/central/com/aliyun/drc/client/2.2.1.39-cloud/client-2.2.1.39-cloud.pom?Expires=1535437088&OSSAccessKeyId=LTAIfU51SusnnfCC&Signature=8kHcUHQ36Mrmx9XQZHsMra%2BllaU%3D
```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- delete for public maven deploy -->
    <!--<parent>
        <groupId>com.taobao</groupId>
        <artifactId>parent</artifactId>
        <version>1.0.3</version>
    </parent>-->

    <name>client</name>
    <url>https://www.aliyun.com/product/dts</url>
    <description>The core java client for accessing Data Transmission Service</description>
    <packaging>jar</packaging>
    <groupId>com.aliyun.drc</groupId>
    <artifactId>client</artifactId>
    <version>2.2.1.39-cloud</version>
```


```
# run an nexus-server
mkdir /var/data/tmp-nexus
chown -R 200  /var/data/tmp-nexus
docker run -d -p 8081:8081 --name nexus -e MAX_HEAP=768m -v /var/data/tmp-nexus:/sonatype-work sonatype/nexus


# generate a new project
mvn -B archetype:generate   -DarchetypeGroupId=org.apache.maven.archetypes   -DgroupId=com.mycompany.app   -DartifactId=my-app

# config settings.xml and then build
rm -rf /tmp/.m2/
mkdir -p /tmp/.m2/
m2_r=/tmp/.m2/Repository_a   && mkdir -p ${m2_r} && mvn -Dmaven.repo.local=${m2_r} -X -U -B clean compile -gs settings.xml >aliyun.log 2>&1
m2_r=/tmp/.m2/Repository_142 && mkdir -p ${m2_r} && mvn -Dmaven.repo.local=${m2_r} -X -U -B clean compile -gs settings_2.xml >142.log 2>&1
m2_r=/tmp/.m2/Repository_a_c && mkdir -p ${m2_r} && mvn -Dmaven.repo.local=${m2_r} -X -U -B clean compile -gs settings.aliyun.central.xml >aliyun.central.log 2>&1

```

settings.aliyun.central.xml
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" 
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">

  <localRepository>/tmp/.m2/Repository</localRepository>
  <pluginGroups>
  </pluginGroups>

  <proxies>
  </proxies>

  <servers>
  </servers>

  <mirrors>
    <mirror>
      <id>Nexus</id>
      <name>Nexus_Central</name>
      <url>https://maven.aliyun.com/repository/central</url>
      <mirrorOf>central</mirrorOf>
    </mirror>
  </mirrors>

  <profiles>
        <profile>
                <id>xx-profile</id>
                <activation>
                        <activeByDefault>true</activeByDefault>
                </activation>
                <properties>
        </properties>

                <repositories>
                        <repository>
                                <id>public-repository</id>
                                <url>https://maven.aliyun.com/repository/central</url>
                                <releases>
                                        <enabled>true</enabled>
                                        <updatePolicy>never</updatePolicy>
                                </releases>
                        </repository>
                </repositories>
                <pluginRepositories>
                        <pluginRepository>
                                <id>public-pluginRepository</id>
                                <url>https://maven.aliyun.com/repository/central</url>
                                <releases>
                                        <enabled>true</enabled>
                                        <updatePolicy>never</updatePolicy>
                                </releases>
                        </pluginRepository>
                </pluginRepositories>
        </profile>

  </profiles>

```
