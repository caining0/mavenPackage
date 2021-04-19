

## Android组件化实战之利用Maven优雅地调试SDK-竹叶青

## 打包sdk到Maven仓库

[Maven](https://maven.apache.org/) 是 Apache 下的一个纯 Java 开发的开源项目，基于项目对象模型（缩写：POM）概念，Maven利用一个中央信息片断能管理一个项目的构建、报告和文档等步骤。Maven 也可被用于构建和管理各种项目，例如 C#，Ruby，Scala 和其他语言编写的项目。

### POM介绍

POM（Project Object Model）即项目对象模型，通过xml格式保存的pom.xml文件，作用类似ant的build.xml文件，功能更强大。该文件用于管理：源代码、配置文件、开发者的信息和角色、问题追踪系统、组织信息、项目授权、项目的url、项目的依赖关系等等；

[Glide：POM举个栗子](https://repo1.maven.org/maven2/com/github/bumptech/glide/glide/4.12.0/glide-4.12.0.pom)

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">  
  <modelVersion>4.0.0</modelVersion>  
  <groupId>com.github.bumptech.glide</groupId>  
  <artifactId>glide</artifactId>  
  <version>4.12.0</version>  
  <packaging>aar</packaging>  
  <name>Glide</name>  
  <description>A fast and efficient image loading library for Android focused on smooth scrolling.</description>  
  <url>https://github.com/bumptech/glide</url>  
  ...
</project>
```

### POM节点

- `groupId` : 组织标识，例如：com.github.bumptech.glide，在目录下，将是: com/github/bumptech/glide目录。
- `artifactId` : 项目名称，例如：Glide。
- `version` : 版本号。
- `packaging` : 打包的格式，可以为：pom , jar , maven-plugin , ejb , war , ear , rar , par ,aar等。

### Maven的目标
- 简化构建过程
- 提供统一的构建系统
- 提供优质的项目信息
- 鼓励更好的开发实践

### Maven仓库

Maven 仓库能帮助我们管理构件（主要是JAR），它就是放置所有JAR文件（WAR，ZIP，POM，AAR等等）的地方。

举个栗子：[阿里](http://maven.aliyun.com/nexus/content/groups/public)&[jcenter](https://mvnrepository.com/repos/jcenter)& [maven 仓库](https://mvnrepository.com/) 

Android引入仓库 : 根 build.gradle配置 [gradle—doc](https://docs.gradle.org/current/dsl/org.gradle.api.plugins.ExtraPropertiesExtension.html#org.gradle.api.plugins.ExtraPropertiesExtension)

1. buildscript repositories vs 2.allprojects repositories

```groovy
 buildscript {
   repositories {
       		jcenter()
          google()
      }
    }
allprojects {
    addRepos(repositories)
    repositories{
    	 jcenter()
       google()
       mavenLocal()
     	 mavenCentral()
       maven { url "http://maven.aliyun.com/nexus/content/groups/public" }
       maven { url "file:///${rootProject.getRootDir()}/asdk" }
       maven { url "file:///${rootProject.getRootDir()}/bsdk" }
    }
}
```

> `buildscript` { }：Configures the build script classpath for this project.

> `allprojects` { }：Configures this project and each of its sub-projects.

注解：`buildscript`是gradle脚本执行所需依赖，`allprojects`是项目本身需要的依赖，所以我们将自己所需要的仓库加到allprojects配置中

通过以上我们知道，打包sdk 需要一个托管的仓库，那么下面我们就介绍一下，如何搭建一个私有仓库；基于安全考虑不用公共仓库，学习一下如何搭建私有仓库。

### 通过nexus搭建私有仓库

[Nexus](https://www.sonatype.com/nexus/repository-oss-download) 简介：是Maven仓库管理器，如果你使用Maven，你可以从[Maven中央仓库](https://repo1.maven.org/maven2/) 下载所需要的构件（artifact）

简介2：**World’s #1 Repository Manager**（自己夸自己世界第一，其他吹牛逼的不用看了）Single source of truth for all of your components, binaries, and build artifacts Efficiently distribute parts and containers to developers Deployed at more than 100,000 organizations globally

搭建Maven私有仓库步骤 

#### 1.下载

 https://www.sonatype.com/download-oss-sonatype

#### 2.启动  

```
~ » /Users/caining/Documents/applications/nexus-3.28.1-01-mac/nexus-3.28.1-01/bin/nexus start
Starting nexus
```

#### 3.访问

http://localhost:8081/

#### 4.修改端口方法

```
cd /Users/caining/Documents/applications/nexus-3.28.1-01-mac/nexus-3.28.1-01/etc
vim nexus-default.properties

###########################配置如下

# Jetty section
application-port=8081
application-host=0.0.0.0
```

<img src="https://www.hualigs.cn/image/606c094878184.jpg" alt="image-20210326202528170" style="zoom:80%;" />

#### 5.登录 

nexus 2 的默认账号密码 admin：默认密码admin123登录； nexus 3第一次启动后  nexus3 会自动生成admin 的密码，存在admin.password

<img src="https://www.hualigs.cn/image/606c0965e64ef.jpg" alt="image-20210326205307574" style="zoom:80%;" />

```
查看密码
~ » vim /Users/caining/Documents/applications/nexus-3.28.1-01-mac/sonatype-work/nexus3/admin.password
```

3866aa0a-39e2-4e14-90bb-4a31c6c303e1

<img src="https://www.hualigs.cn/image/606c0a6e4962e.jpg" alt="image-20210326210024703" style="zoom:50%;" />

#### 6.首次登录后设置密码

#### 7.停止

```
~ » /Users/caining/Documents/applications/nexus-3.28.1-01-mac/nexus-3.28.1-01/bin/nexus stop
Shutting down nexus
Stopped.
```

### gradle 脚本打包sdk并上传Maven repository

一、在工程下新建 maven-push.gradle 脚本

```groovy
apply plugin: 'maven'

uploadArchives {
      repository(url: maven.address) { authentication(userName: maven.username, password: maven.password) }
      pom.groupId = maven.MAVENGROUPID
      pom.artifactId = ARTIFACT
      pom.version = maven.appLibVersion
      pom.packaging = 'aar'
      pom.project {
                licenses {
                    license {
                        name 'The Apache Software License, Version 2.0'
                        url 'http://www.apache.org/licenses/LICENSE-2.0.txt'
                    }
                }
            }
}
```

二、配置根gralde配置

```groovy
1.新建 config.gradle
2.apply from: 'config.gradle'
3.
ext {
		  maven = [
            address         : "http://localhost:8081/repository/maven-releases/",//maven 仓库地址
            username        : "admin",
            password        : "123456",
            appLibVersion   : "1.0.6",//打包版本号
            buildMAVEN      : true ,//是否打开upload功能，为宿主工程准备、、最小侵入宿主工程
            MAVENGROUPID      : "com.cnn.sdk"//打包groupId 这里统一设置，如需要不同，请module中单独设置
    ]
}
```

三 、gradle [ext](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.gradle.org%2Fcurrent%2Fdsl%2Forg.gradle.api.plugins.ExtraPropertiesExtension.html%23org.gradle.api.plugins.ExtraPropertiesExtension) 是什么？

1. ext属性是ExtensionAware类型的一个特殊的属性，本质是一个Map类型的变量；
2. 使用ext属性的好处可以在任何能获取到rootProject的地方访问这些属性 
3. gradle.properties 与 ext 的不同？由于ext 结构是map，所以可以分组，比 gradle.properties 好用一些，一些系统级的配置可以放gradle.properties ，用户基本的配置可以放ext 比如版本号啥的。

四、module 子 gradle.properties 增加artifact名字

1. 新建module gradle.properties
2. ARTIFACT= artifact—name // 这个不能随意起，起名规则后面讲

五、sdk-module 子 build.gradle 增加配置

```groovy
try {
    if (maven.buildMAVEN){//这里是true  在宿主工程里，会访问不到而报错
        apply from: '../maven-push.gradle'
    }
}catch (all){
}
```

以上，执行 uploadArchives task 就会打包并upload 到仓库

```bash
> Task :module01:uploadArchives
> Task :module02:writeReleaseAarMetadata
> Task :module02:bundleReleaseAar
> Task :module02:uploadArchives

Deprecated Gradle features were used in this build, making it incompatible with Gradle 7.0.
Use '--warning-mode all' to show the individual deprecation warnings.
See https://docs.gradle.org/6.5/userguide/command_line_interface.html#sec:command_line_warnings

BUILD SUCCESSFUL in 781ms
43 actionable tasks: 41 executed, 2 up-to-date
10:46:48 AM: Task execution finished 'uploadArchives'.
```

<img src="https://www.hualigs.cn/image/606c0a6e4a041.jpg" alt="image-20210330105751560" style="zoom:50%;" />



Module01 添加依赖implementation project(path: ':module02')，这时候再执行uploadArchives。

**重点：抛出问题：如果模块间依赖，以上脚本在全量打包时会报错 如下**

![image-20210330123054236](https://www.hualigs.cn/image/606c0a6deb7c3.jpg)

**原因：module01 依赖是本地包，所以报错**

![image-20210330134953074](https://www.hualigs.cn/image/606c0a6ded606.jpg)

**如何解决？** 

1. 思路1：先传module02 再修改module01依赖为线上包，再传module01 ，这个思路可行，但如果遇到module 过多，执行起来过于麻烦，这里不再演示，感兴趣的同学可自己执行。

### 相互依赖打包优化

1. 问题 ：填坑对于有互相依赖的包如何处理？对于关联依赖 sdk 如何一键 build 并上传Maven仓库

2. 以上存在的问题 ：以上脚本虽然初步完成了sdk 打包并上传Maven仓库，但有问题存在，加入sdk 子module 关联依赖时，子包不发布，主包无法打包成功，那么除了手动一步一步上传外，有没有办法一键上传？

3. 优化思路：动态替换打包后groupId 和 version（由于是动态替换，这里的工程moudle 的名称应该与gradle.properties配置的artifact 保持一致）
4. 关键脚本如下：

```groovy
 pom.withXml {
                def childs = asNode().children()
                childs.each {
                    child ->
                        print("childName->"+child.name().toString()+"\n")
                        if (child.name().toString().contains('dependencies')) {
                            child.children().each {
                                print("dependencies->"+it.text().toString()+"\n")
                                if (it.text().toString().contains("unspecified")) {
                                    def iterator = it.children().iterator()
                                    def remove = false
                                    while (iterator.hasNext()) {
                                        def node = iterator.next()
                                        if (node.name().toString().contains('groupId')) {
                                            iterator.remove()
                                        }
                                        if (node.name().toString().contains('version')) {
                                            iterator.remove()
                                            remove = true
                                        }
                                    }
                                    if (remove) {
                                        it.appendNode('version', maven.appLibVersion)
                                        it.appendNode('groupId', maven.MAVENGROUPID)
                                    }
                                    print("\n return ->"+it.text().toString()+"\n")
                                }
                            }

                        }
                }
            }
```

### 宿主通过仓库引入sdk

引入方式也很简单

```groovy
dependencies {
    implementation fileTree(include: ['*.jar'], dir: 'libs')
    implementation "groupId:artifactId:version"//此处对应上传的maven 仓库的信息
}
```



## 通过宿主工程优雅的调试sdk

通过仓库引入的第三方sdk 在宿主工程里调试，很不方便，虽然可以断电，但源码无法修改，使得在宿主程序中，遇到问题，查问题时，非常不方便，那么有没有好的办法，在本地开发时，不使用仓库包，而是直接引用源码呢？上线时，通过修改配置，引用远程仓库。

### 通过gradle脚本控制引用源码or仓库

我们知道android module引入模块，是在settings.gradle中，如下：

### Module常规引入

```groovy
include ':sdka'
```

### 参数控制动态引入

![image-20210401135541636](https://www.hualigs.cn/image/606c0a6decec6.jpg)

1. 宿主根gradle.properties 增加控制参数boolean useLocalLib = true/false

2. 通过settings.gradle配置动态控制是否使用仓库

```groovy
if (useLocalLib.toBoolean()){
    print("sdk调试使用，线上环境忽略以下内容")
    include(':module01')
    include(':module02')
    print("path---->"+getRootDir().getParent())
    project(':module01').projectDir = new File(getRootDir().getParent(),"/MavenPackage/module01")
    project(':module02').projectDir = new File(getRootDir().getParent(),"/MavenPackage/module02")
}
```

3. 同理，宿主工程引入lib时 ，用gradle.properties-useLocalLib字段控制引用仓库or本地module

```groovy
dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    if (useLocalLib.toBoolean())
    	implementation project(":module01")
    else
    	implementation 'com.cnn.sdk:module01:1.0.0'
}
```

以上，可以在宿主工程里，通过参数userLocalSDK，达到引入sdk源码的目的，源码并不在宿主git 下；完成优雅的调试sdk 的目的。

## 总结

- 通过Maven仓库引入的好处：组件隔离，业务隔离，减少侵入等

- 使用Nexus搭建Maven 私有仓库，以及常用命令 初始密码等，以及使用

- 通过gradle plugin: 'maven' 打包并upload to Maven Repository

- 宿主工程引用时，动态控制指向，是否打开源码，达到调试sdk源码的目的，并且git分