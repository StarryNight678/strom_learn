# Storm编译运行

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

编译运行:
初始化安装storm所需依赖：$ 
mvn clean install -DskipTests=true


使用Maven打包storm拓扑：$ mvn package
搭建好运行环境并提交：
