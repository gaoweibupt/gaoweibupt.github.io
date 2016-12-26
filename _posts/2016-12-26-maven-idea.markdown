---
lyout:      post
title:      "idea下新建maven工程"
subtitle:   "maven使用详解"
date:       2016-12-26 12:00:00
author:     "galway"
header-img: "img/home-bg-o.jpg"
catalog:  ture
tags:
    - engineering
    - maven
---

昨天在idea客户端下使用maven新建工程，遇到了许多问题，所以纪录一下详细的过程。

### 安装maven

idea自带的maven有很多问题，例如版本过旧、在idea的命令行中不能使用等问题。

1. 安装maven
- 到[maven官网](http://maven.apache.org/)下载maven文件
- 将maven解压到/usr/local/maven目录下
- 在文件`~/.bash_profile`中配置环境变量

    ```
    export M2_HOME=/usr/local/maven/apache-maven-3.3.9
    export PATH=$PATH:$M2_HOME/bin
    ```
__注意__: 由于安装了zsh，所以在~/.zshrc 最后添加了`source ~/.bash_profile`

2. idea中配置maven
file->Other Settings->Default Setting中，将maven的home directory设置成下载的maven路径，或者
![img](/img/post_img/maven_idea.png)

另外，需要配置maven的setting.xml文件，将其下载的镜像改成国内的阿里镜像。

```
  <mirrors>
     <mirror>
        <id>alimaven</id>
        <mirrorOf>central</mirrorOf>
        <name>aliyun maven</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
        <!--<url>http://mvnrepository.com/</url>-->
    </mirror>
  </mirrors>
```

### archetype创建工程 

maven使用archetype创建工程时，可以根据不同的archetype生成工程目录。如果网络不好的话，使用archetype的`mvn generate` 通常会卡在`[INFO]Generating project in Batch mode`的情况下，通过`mvn generate -X`可以看到，此时在网络中正在寻找一个xml文件。
为了加快速度，不使用在网络中查找的这种方式，使用`internal`的模式可以加快目录生成，只需要在创建工程时，将`archetypeCatalog`的属性设置为`internal`即可。
 ![img](/img/post_img/maven_config.png)

创建成功后，idea的maven会自动生成文件目录，或者自己手动在终端中执行`mvn generate`

__另外__，由于maven初次使用，可能工程的plugin 依赖有问题，因此在pom中添加了如下内容，解决了不能成功`mvn install`的问题

```
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-resources-plugin</artifactId>
        <version>2.4.1</version>
      </plugin>

      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-clean-plugin</artifactId>
        <version>2.4.1</version>
      </plugin>
    </plugins>
  </build>
```
