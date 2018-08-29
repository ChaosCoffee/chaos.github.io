---
title: Maven-deploy配置方法
comments: false
date: 2018-08-20 16:49:39
categories: tools
tags: Maven
img:
---
# Maven-deploy配置方法
<!-- TOC -->

- [Maven-deploy配置方法](#maven-deploy%E9%85%8D%E7%BD%AE%E6%96%B9%E6%B3%95)
    - [deploy解释](#deploy%E8%A7%A3%E9%87%8A)
    - [setting.xml配置](#settingxml%E9%85%8D%E7%BD%AE)
    - [pom.xml配置](#pomxml%E9%85%8D%E7%BD%AE)

<!-- /TOC -->
## deploy解释
---
deploy顾名思义，就是上传的意思。在本地的pom文件配置好之后，执行deploy命令，可以将maven所打的jar包上传到远程的repository，便于其他开发者和工程共享


## setting.xml配置
---  

``` xml
<server>
    <id>releases</id>
    <username>admin</username>
    <password>admin123</password>
</server>
<server>
    <id>Snapshots</id>
    <username>admin</username>
    <password>admin123</password>
</server>
```

## pom.xml配置  
--- 

``` xml
<distributionManagement>  
   <repository>  
     <id>releases</id>  
     <name>Internal Releases</name>  
     <url>http://localhost:8081/nexus/content/repositories/releases</url>  
   </repository>  
   <snapshotRepository>  
     <id>Snapshots</id>  
     <name>Internal Releases</name>  
     <url>http://localhost:8081/nexus/content/repositories/snapshots</url>  
   </snapshotRepository>
 </distributionManagement> 
```

说明: `id`和`url`务必写正确，否则无法正确deploy，这个步骤需要验证用户信息，所以与上一步骤`server` `id`对应上

