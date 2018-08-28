
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
