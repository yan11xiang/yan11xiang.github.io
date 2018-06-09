---
layout:     post
title:      "SPRINGBOOT开发实战(1)"
subtitle:   ""
date:       2018-06-01 12:00:00
author:     "闫祥"
header-img: "img/spring/spring-boot.jpeg"
tags:       SPIRNG-BOOT
---

## Spring boot 开发实战

### 缘起
因为项目准备用Spring boot搭建，所以处理完手里的工作，开始了解准备Springboot。
虽然在此之前有接触过，但是仅仅是运行了下官方的demo，对于一个生产环境的架构，我在搭建完成之后记录下来。

### 尝试 Spring boot
使用 idea 自动创建一个 Spring boot 的 demo 运行下。  
new project --> Spring Initializr  --> 选择默认即可 --> next --> 配置项目名称等信息 --> next --> 可以选择一些依赖项目(比如 jdbc，mysql test，thymeleaf 等), 然后就等着 idea 自动化构造好一个 Spring boot 项目吧。  
当然也可以从 https://start.spring.io/ 下载，自己解压，在 idea 中导入也是可以的。

运行 DemoApplication 的main 方法，我们就可以启动一个Spring boot 程序了。
```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.0.2.RELEASE)

```
### 生产项目结构

项目成员中有考虑部分使用kotlin开发，增加了kotlin 和 java 的混合结构。项目的maven目录结构如下图，如果不需要kotlin 的去掉即可。

``` shell
├── demo-admin
│   ├── pom.xml
│   └── src
│       ├── main
│       │   ├── java
│       │   ├── kotlin
│       │   └── resources
│       │       ├── application.properties
│       │       ├── static
│       │       └── templates
│       └── test
│           ├── java
│           ├── kotlin
│           └── resources
│               └── application.properties
├── demo-common
│   ├── pom.xml
│   └── src
│       ├── main
│       │   ├── java
│       │   └── kotlin
│       └── test
│           ├── java
│           ├── kotlin
│           └── resources
│               └── application.properties
├── demo-front
│   ├── pom.xml
│   └── src
│       ├── main
│       │   ├── java
│       │   ├── kotlin
│       │   └── resources
│       │       ├── application.properties
│       │       ├── static
│       │       └── templates
│       └── test
│           ├── java
│           ├── kotlin
│           └── resources
│               └── application.properties
└── pom.xml

```

### 项目文件

#### root pom.xml 文件

** 如果不需要kotlin，移除即可 **

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.demo</groupId>
    <artifactId>demo-root</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>pom</packaging>

    <name>demo-root</name>
    <description>demo</description>

    <modules>
        <module>demo-common</module>
        <module>demo-front</module>
        <module>demo-admin</module>
    </modules>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <kotlin.version>1.2.31</kotlin.version>
        <kotlin.compiler.incremental>true</kotlin.compiler.incremental>
        <springboot.version>2.0.2.RELEASE</springboot.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${springboot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>javax.persistence</groupId>
                <artifactId>javax.persistence-api</artifactId>
                <version>2.2</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>${springboot.version}</version>
            </plugin>
            <plugin>
                <!-- kotlin 打包插件，java kotlin 混合编译 -->
                <artifactId>kotlin-maven-plugin</artifactId>
                <groupId>org.jetbrains.kotlin</groupId>
                <version>${kotlin.version}</version>
                <configuration>
                    <jvmTarget>1.8</jvmTarget>
                </configuration>
                <executions>
                    <execution>
                        <id>compile</id>
                        <goals>
                            <goal>compile</goal>
                        </goals>
                        <configuration>
                            <sourceDirs>
                                <sourceDir>${project.basedir}/src/main/kotlin</sourceDir>
                                <sourceDir>${project.basedir}/src/main/java</sourceDir>
                            </sourceDirs>
                        </configuration>
                    </execution>
                    <execution>
                        <id>test-compile</id>
                        <goals>
                            <goal>test-compile</goal>
                        </goals>
                        <configuration>
                            <sourceDirs>
                                <sourceDir>${project.basedir}/src/test/kotlin</sourceDir>
                                <sourceDir>${project.basedir}/src/test/java</sourceDir>
                            </sourceDirs>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.5.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
                <executions>
                    <!-- Replacing default-compile as it is treated specially by maven -->
                    <execution>
                        <id>default-compile</id>
                        <phase>none</phase>
                    </execution>
                    <!-- Replacing default-testCompile as it is treated specially by maven -->
                    <execution>
                        <id>default-testCompile</id>
                        <phase>none</phase>
                    </execution>
                    <execution>
                        <id>java-compile</id>
                        <phase>compile</phase>
                        <goals>
                            <goal>compile</goal>
                        </goals>
                    </execution>
                    <execution>
                        <id>java-test-compile</id>
                        <phase>test-compile</phase>
                        <goals>
                            <goal>testCompile</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
    <distributionManagement>
        <repository>
            <id>releases</id>
            <name>Nexus Releases Repository</name>
        </repository>
        <snapshotRepository>
            <id>snapshots</id>
            <name>Nexus Snapshots Repository</name>
        </snapshotRepository>
    </distributionManagement>

</project>
```

#### admin front common 模块
我们用上面使用 idea 创建好的工程简单修改一下。
pom 文件修改一下，增加 parent 标签即可
``` xml
    <parent>
        <groupId>com.demo</groupId>
        <artifactId>demo-root</artifactId>
        <version>1.0.0-SNAPSHOT</version>
    </parent>
```

对于admin 和 front ,我们还要修改 DemoApplication 修改为 AdminAppliacation 和 FrontApplication 即可


** 这样一个 前后台和common模块的纯后端的工程即搭建完成。 ** 

导入工程到 idea 中，我们启动一下AdminApplication 的main方法,检测下是否可以成功启动。


> 参考资料：  
> [spring-io](https://spring.io/guides/gs/intellij-idea/)
>

*****
[记录并分享自己的学习与成长](http://cbrothercoder.com/)


