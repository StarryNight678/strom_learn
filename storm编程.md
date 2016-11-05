## Storm编译运行

## maven使用

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

- 入门文档

[mvn官方入门文档](http://maven.apache.org/guides/getting-started/maven-in-five-minutes.html)

1. 创建工程

```
mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=my-app -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false

org.apache.storm.starter

```

目录结构

  ./pom.xml
  ./src
  ./src/test
  ./src/test/java
  ./src/test/java/com
  ./src/test/java/com/mycompany
  ./src/test/java/com/mycompany/app
  ./src/test/java/com/mycompany/app/AppTest.java
  ./src/main
  ./src/main/java
  ./src/main/java/com
  ./src/main/java/com/mycompany
  ./src/main/java/com/mycompany/app
  ./src/main/java/com/mycompany/app/App.java


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

## 性能测试

[yahoo test](https://github.com/yahoo/storm-perf-test/)


