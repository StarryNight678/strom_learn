## Storm编译运行

## 1maven依赖下载

[Maven教程](http://wiki.jikexueyuan.com/project/maven/build-life-cycle.html)
- 更换镜像

[ubuntu eclipse 安装maven插件](http://askubuntu.com/questions/141204/what-is-the-correct-way-to-install-maven-and-eclipse)


更换 Maven 镜像Maven 的官方镜像比较慢，建议使用其他网站提供的镜像，速度比较快

国内访问repo1.maven.org访问不了，导致maven不能下载依赖，解决方法是自己设置maven的mirrors，就是设置镜像：

在~/.m2/目录下建立一个settings.xml文件，内容如下

```
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"  
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">  
  <mirrors>
  	<!-- mirror | Specifies a repository mirror site to use instead of a given 
  	repository. The repository that | this mirror serves has an ID that matches 
  	the mirrorOf element of this mirror. IDs are used | for inheritance and direct 
  	lookup purposes, and must be unique across the set of mirrors. | -->
  	<mirror>
	  	<id>id</id>
	  	<mirrorOf>central</mirrorOf>
	  	<name>name</name>
	  	<url>http://XXX</url>
  	</mirror>
  </mirrors>
</settings>
```

最终发现,该镜像速度较快
```
<mirror>  
    <id>uk</id>  
    <mirrorOf>central</mirrorOf>  
    <name>Human Readable Name for this Mirror.</name>  
    <url>http://uk.maven.org/maven2/</url>  
</mirror>
```


## 出现构建项目时间长问题
加上参数
```
-Dversion=1.0 -DarchetypeCatalog=internal
```

## 2入门文档

[mvn官方入门文档](http://maven.apache.org/guides/getting-started/maven-in-five-minutes.html)

1. 创建工程

```
mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=my-app -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false

org.apache.storm.starter

mvn archetype:generate -DgroupId=org.apache.storm.starter -DartifactId=test -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```


maven使用 参考[Apache Maven 入门篇](http://www.oracle.com/technetwork/cn/community/java/apache-maven-getting-started-1-406235-zhs.html)

创建maven项目
```
mvn archetype:generate -DgroupId=com.mycompany.helloworld -DartifactId=helloworld -Dpackage=com.mycompany.helloworld -Dversion=1.0-SNAPSHOT
```


1. archetype:generate  创建项目类型 archetype 是一个插件的名字，generate是目标(goal)的名字。这个命令的意思是告诉 maven 执行 archetype 插件的 generate 目标。插件目标通常会写成 pluginId:goalId
1. Archetype 可以理解成项目的模型。 包括从简单的 Swing 到复杂的 Web 应用
1. DartifactId: 项目名字

构建程序
```
mvn package 
```

第一次运行时,会将需要的包下载到本地.Maven 默认的本地库是` ~/.m2/repository/` ，在 Windows 下是 `%USER_HOME%\.m2\repository\` 。

测试程序
```
java -cp target/helloworld-1.0-SNAPSHOT.jar com.mycompany.helloworld.App
```
-cp 和 -classpath 一样，是指定类运行所依赖其他类的路径，通常是类库，jar包之类，需要全路径到jar包.

Maven - 构建生命周期

|阶段 | 处理 |  描述|
|prepare-resources |资源拷贝 | 本阶段可以自定义需要拷贝的资源
|compile |编译  |本阶段完成源代码编译
|package |打包  |本阶段根据 pom.xml 中描述的打包配置创建 JAR / WAR 包
|install |安装  |本阶段在本地 / 远程仓库中安装工程包

删除构建目录

  mvn clean





## 3pom.xml例子

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.mycompany.helloworld</groupId>
  <artifactId>helloworld</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>helloworld</name>
  <url>http://maven.apache.org</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>

```

## 4Maven 库

当第一次运行 maven 命令的时候，你需要 Internet 连接，因为它要从网上下载一些文件。那么它从哪里下载呢？它是从 maven 默认的远程库(http://repo1.maven.org/maven2) 下载的。这个远程库有 maven 的核心插件和可供下载的 jar 文件。

但是不是所有的 jar 文件都是可以从默认的远程库下载的，比如说我们自己开发的项目。这个时候，有两个选择：要么在公司内部设置定制库，要么手动下载和安装所需的jar文件到本地库。

本地库是指 maven 下载了插件或者 jar 文件后存放在本地机器上的拷贝。在 Linux 上，它的位置在 ~/.m2/repository，在 Windows XP 上，在 C:\Documents and Settings\username\.m2\repository ，在 Windows 7 上，在 C:\Users\username\.m2\repository。当 maven 查找需要的 jar 文件时，它会先在本地库中寻找，只有在找不到的情况下，才会去远程库中找。

运行下面的命令能把我们的 helloworld 项目安装到本地库：

     $mvn install

一旦一个项目被安装到了本地库后，你别的项目就可以通过 maven 坐标和这个项目建立依赖关系。比如如果我现在有一个新项目需要用到 helloworld，那么在运行了上面的 mvn install 命令后，我就可以如下所示来建立依赖关系：

Xml 代码

    <dependency>
      <groupId>com.mycompany.helloworld</groupId>
      <artifactId>helloworld</artifactId>
      <version>1.0-SNAPSHOT</version>
    </dependency> 

好了，maven 的核心概念就简单的介绍到这里。至于在 Eclipse 中如何使用 maven，这个网上很多了，google 一下就行。


编译运行:
初始化安装storm所需依赖：
``` 
mvn clean install -DskipTests=true
```

使用Maven打包storm拓扑：
```
mvn package
```
编译
```
mvn compile
```

## Storm 编译

依赖
```
<dependency>
  <groupId>storm</groupId>
  <artifactId>storm</artifactId>
  <version>1.0.2</version>
</dependency>
```

## 5storm运行

1. storm jar 提交
1. storm kill 关闭
1. storm deactivate topology_name  停止发送tuple
1. storm activate topology_name    恢复发送tuple
1. storm rebalancce topology_name  
1. storm remoteconfvalue topology_name  查看公共参数




storm.blueprints.chapter1.v3.WordCountTopology  WordCountTopology


## storm Jstorm  Heron 性能测试

- storm

storm jar  ./target/performance-test-strom-1.0.jar   com.alibaba.jstorm.performance.test.FastWordCountTopology


storm deactivate  FastWordCountTopology
storm activate FastWordCountTopology
storm kill FastWordCountTopology

- heron

heron submit local com.alibaba.jstorm.performance.test.FastWordCountTopology FastWordCountTopology

heron submit --verbose local performance-test-heron-1.0.jar com.alibaba.jstorm.performance.test.FastWordCountTopology  FastWordCountTopology  --deploy-deactivated

heron submit local \
~/.heron/examples/heron-examples.jar \
com.twitter.heron.examples.ExclamationTopology \
ExclamationTopology \
--deploy-deactivated



heron submit local  \
./target/performance-test-heron-1.0.jar  \
com.alibaba.jstorm.performance.test.PerformanceTestTopology  \
PerformanceTestTopology  \
--deploy-deactivated


heron submit local performance-test-heron-1.0.jar com.alibaba.jstorm.performance.test.PerformanceTestTopology simple.yaml
heron submit local performance-test-heron-1.0.jar com.alibaba.jstorm.performance.test.FastWordCountTopology simple.yaml 


heron submit local  \
./target/performance-test-heron-1.0.jar  \
com.alibaba.jstorm.performance.test.PerformanceTestTopology  \
PerformanceTestTopology  \


java -jar /tmp/storm-starter.jar ExclamationTopology


src/main/java/com/alibaba/jstorm/performance/test/PerformanceTestTopology.java
src/main/java/com/alibaba/jstorm/performance/test/WordCountTopology.java
src/main/java/com/alibaba/jstorm/performance/test/FastWordCountTopology.java


heron submit local  \
./target/performance-test-heron-1.0.jar  \
com.alibaba.jstorm.performance.test.PerformanceTestTopology  \
PerformanceTestTopology  \
--deploy-deactivated

java -jar 


 heron submit local \
~/.heron/examples/heron-examples.jar \
com.twitter.heron.examples.MultiStageAckingTopology \
MultiStageAckingTopology \
--deploy-deactivated


heron activate local MultiStageAckingTopology
heron deactivate local MultiStageAckingTopology
heron kill local MultiStageAckingTopology

## 6性能测试

[yahoo test](https://github.com/yahoo/storm-perf-test/)


