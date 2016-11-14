# 1.前言
## 1.1.编写前言

# 2.Maven的安装与配置
## 2.1.安装前的准备
Maven 是一个项目管理和构建自动化工具，是Apache Fundation下的一个Java项目。常用于Java项目中依赖管理。
1. 环境要求：
- Maven 3.3要求JDK 1.7以上。详细要求参照 [Maven官网](http://maven.apache.org/download.cgi)。
2. 下载Maven  
直接去 [官网下载](http://maven.apache.org/download.cgi)

## 2.2.安装Maven
Maven两种安装包格式：
- 已经编译的二进制包
直接解压到安装目录即可：
```shell
$ cd /path/to/Download
$ tar -xf apache-maven-3.3.9-bin.tar.gz
$ sudo mv apache-maven-3.3.9 /usr/local/
$ sudo ln -s /usr/local/apache-maven-3.3.9/bin/* /usr/bin/
```

- 源码包
源码包的安装与大多数linux软件安装过程一样。

## 2.3.Maven配置


# 3.Maven工程
## 3.1.Maven工程的基本目录结构
Project  
|--src  
|  |--main  
|  |  |--java  
|  |  |  |--hello  
|  |--test  
|  |  |  |--java  
|  |  |  |  |--hello  

其中：
- src：源码目录
- hello: 包
- test: 单元测试

## 3.2.工程样例
### 3.2.1.基本应用
1. 建立基本目录结构
```shell
$ cd /path/to/Projects
$ mkdir hello.aeonluck.me
$ cd hello.aeonluck.me
$ mkdir -p src/main/java/hello
$ mkdir test
```

2. 建立应用类文件
> **注意**：本节代码来自于spring官网。详情可见：[Building Java Projects with Maven](https://spring.io/guides/gs/maven/)

***src/main/java/hello/HelloWorld.java***
```java
package hello;

public class HelloWorld {
    public static void main(String[] args) {
        Greeter greeter = new Greeter();
        System.out.println(greeter.sayHello());
    }
}
```

***src/main/java/hello/Greeter.java***

```java
package hello;

public class Greeter {
    public String sayHello() {
        return "Hello world!";
    }
}
```

3. 建立工程描述文件pom.xml
> 小提示：POM：Project Object Model。

pom文件内容如下：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.springframework</groupId>
    <artifactId>gs-maven</artifactId>
    <packaging>jar</packaging>
    <version>0.1.0</version>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.1</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <transformers>
                                <transformer
                                    implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass>hello.HelloWorld</mainClass>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

> pom.xml中节点说明:   
> - groupId : 应用所属团队或单位的id，通常为域名反序。 
> - artifactId : 工程标识，如当前应用打包后的jar/war名。
> - version: 当前应用版本号
> - packaging : 打包格式，默认为jar，也可以选择war包。

4. 构建工程
- 编译及解决依赖问题
```shell
$ cd /path/to/hello.aeonluk.me
$ mvn compile
```
此时maven会首先查询 `pom.xml` 引入的包，查找本地仓库是及工程目录，如不存在相应的包就会从maven的远程apache仓库下载。

- 打包工程：
```shell
mvn package
```
Maven将在工程目录（这里是：hello.aeonluck.me/targe/）下生成一个名为 **gs-maven-0.1.0.jar** 的jar包，同时也会自动生成其它打包所需的目录与文件。

- 测试jar包
```shell
java -jar /path/to/gs-maven-0.1.0.jar
```
输出 `Hello World!` 即表示正常运行。

5. 发布包到本地仓库
```shell
$ mvn install
```

### 3.2.2.引入第三方包
1. 修改***src/main/java/hello/HelloWorld.java***，内容如下：
```java
package hello;

import org.joda.time.LocalTime;

public class HelloWorld {
	public static void main(String[] args) {
		LocalTime currentTime = new LocalTime();
		System.out.println("The current local time is: " + currentTime);
		Greeter greeter = new Greeter();
		System.out.println(greeter.sayHello());
	}
}
```

2. 声明依赖
在pom中加入台下内容：
```xml
<project>
...
<dependencies>
		<dependency>
			<groupId>joda-time</groupId>
			<artifactId>joda-time</artifactId>
			<version>2.2</version>
		</dependency>
</dependencies>
...
</project>
```

3. 重新编译及打包
再次编译时，Maven会重新扫描一次pom文件，从面发现最新的依赖变更，并从仓库中下载相应的依赖包。
```shell
$ cd /path/to/hello.aeonluck.me
$ mvn compile
$ mvn package
```

4. 测试改动后的jar包
```shell
$ java -jar /path/to/gs-maven-0.1.0.jar
``` 

### 3.2.3.单元测试
1. 声明依赖
```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>
```

2. 建立测试类  

***src/test/java/hello/GreeterTest.java***
```java
package hello;

import static org.hamcrest.CoreMatchers.containsString;
import static org.junit.Assert.*;

import org.junit.Test;

public class GreeterTest {

	private Greeter greeter = new Greeter();

	@Test
	public void greeterSaysHello() {
		assertThat(greeter.sayHello(), containsString("Hello"));
	}

}
```

3. 编译、打包及测试
> 小提示：单元测试将会执下 `test/java` 下所有的 `*Test` 文件。

```
$ mvn test
```

# 4.IDE中Maven的使用


# 5.Nexus OSS
## 5.1.Nexus OSS简介
Nexus是一个强大的Maven仓库管理器，它极大地简化了自己内部仓库的维护和外部仓库的访问。利用Nexus你可以只在一个地方就能够完全控制访问 和部署在你所维护仓库中的每个Artifact。Nexus是一套“开箱即用”的系统不需要数据库，它使用文件系统加Lucene来组织数据。Nexus 使用ExtJS来开发界面，利用Restlet来提供完整的REST APIs，通过m2eclipse与Eclipse集成使用。Nexus支持WebDAV与LDAP安全身份认证。
Nexus分为专业收费的Nexus Pro与免费的Nexus OSS两个版本。本文以Nexus OSS为主体。

## 5.2.安装与配置
1. 下载Nexus OSS
**Nexus OSS 3.0** 已经不支持Maven格式，所以这里我们下载 **2.x** 版本。
下载地址：[Nexue OSS下载](https://www.sonatype.com/download-oss-sonatype)

2. 安装Nexus OSS
```shell
$ cd /path/to/Downloads
$ tar -xf nexus-2.14.1-01-bundle.tar.gz
$ cd nexus-2.14.1-01-bundle 
$ sudo mv nexus-2.14.1-01 /usr/local/

# 建立仓库存放目的
$ sudo mkdir /Data
$ sudo mv sonatype-work /Data/

# 建立bin运行软链
# sudo ln -s /usr/local/nexus /usr/local/nexus-2.14.1-01
# sudo ln -s /usr/loca/nexus-2.14.1-01/bin/nexus /usr/local/bin/
```

3. 配置Nexus OSS
```shell
$ cd /usr/local/nexus-2.14.1-01/conf
$ sudo vim nexus-propertities

# 修改以下内容
nexus-work=/Data/nexus-repos/sonatype-work/nexus
```

4. 测试运行
```shell
$ nexus start

# 停止
$ nexus stop

# 重启
$ nexus restart
```
浏览器打开`http://localhost:8081/nexus`，即可看到nexus oss的欢迎界面。

> 小提示：登录用户：admin，默认密码：admin123，登录后可在个人中心修改。

5. nginx反代方式
- 建立vhost
```shell
$ cd /path/to/nginx/conf/vhosts
$ sudo vim nexus.local.me.conf

# 内容如下：
server                                                                             
{                                                                                  
    server_name nexus.aeonluck.me;                                                 
                                                                                   
    location / {                                                                   
        proxy_pass   http://127.0.0.1:8081;                                        
                                                                                   
        proxy_redirect  off;                                                       
        proxy_set_header Host $host;                                               
        proxy_set_header X-Real-IP $remote_addr;                                   
        proxy_set_header X-Forwarded-For                                           
        $proxy_add_x_forwarded_for;                                                
   }                                                                               
}                                                                                    
```

- 添加本地host
```
# sudo vim /etc/hosts

# 增加以下内容
127.0.0.1 nexus.local.me
```

- 测试反代
```shell
$ sudo nginx -s reload
```
浏览器打开`http://nexus.local.me/nexus`即可见到nexus oss欢迎界面。

## 5.3.仓库配置
Nexus的仓库分为以下几种：
- proxy : 代理仓库，对远程仓库的本地代理。
- hosted :  本地仓库。
- virtual：影子仓库，对proxy或hosted仓库的映射。
- group：仓库组，作为逻辑仓库组对外。

首先，我们要对几个重要仓库的配置进行修改，如 `central`,`Apache Snapshots`等，你也可以自己添加外部仓库。
- 点击左侧 **repositories**，右侧打开仓库列表。
- 选择仓库 **central**，点击下方 **configuration**，找到 **“Download Remote Indexes”**，修改为 **true**，保存。
- 右击仓库列表中 **central**，选择 **Update Indexs**，然后打开左侧管理菜单 **Administration** > **Scheduled Tasks**，即可在计划任务列表中看到刚才建立的“update indexs”任务，这需要一定时间。
- 索引下载完成后，我们即可在仓库的 **Browse Index** 中查看索引，并可进行相应的搜索。

> 小提示：  
> 1. 其它仓库配置可以参照上述过程。  
> 2. 可以通过Nexus OSS左侧菜单 **Administration** > **Logging** 查看相应的日志。
> 3. 关于Nexus OSS的其他管理功能不再累述，只需点击左侧菜单按提示操作即可。


# 6. Maven与Nexus OSS的配合使用
通过第5节，我们已经成功的架设好了自己的Maven私库，下面，我们来让Maven使用我们自己的私库。
1. 修改Maven配置，加入私库
```
$ vim ~/.m2/settings.xml

# servers 内增加我们的私库用户
<server>                                                                       
  <id>snapshots</id>                                                                 
  <username>admin</username>                                                   
  <password>admin123</password>                                                
</server> 

# mirrors增加私库
<mirror>                                                                    
  <id>nexus</id>                                                            
  <name>Local nexus repo</name>                                             
  <url>http://nexus.aeonluck.me/nexus/content/groups/public/</url>          
  <mirrorOf>*</mirrorOf>                                                    
</mirror>
```
> 特别要注意的是：  
> 一定要配置为 `<mirrorOf>*</mirrorOf>`，即让所有的仓库都先经过私库，这样可以将远程下载的包缓存到本地。

> 小提示:  
> 1. Maven安装目录下的 **/conf/settings.xml** 起全局控制作用。
> 2. 用户目下的Maven配置文件，即：**~/.m2/settings.xml** 仅对当前用户用效。
                                                                  

2. 添加个国内仓库
默认的apache的仓库下载速度比较慢，这里选择阿里云的Nexus仓库。

```shell
$ vim ~/.m2/setting2.xml
                        
# 在mirrors中增加以下内容
# 注意，位置于我们的私库之后，否则私服不能缓存到下载的包。                                                    
<mirror>                                                                    
  <id>alimaven</id>                                                         
  <name>aliyun maven</name>                                                 
  <url>http://maven.aliyun.com/nexus/content/groups/public/</url>           
  <mirrorOf>central</mirrorOf>                                              
</mirror>
```
> 关于Maven的mirror:  
> 1. mirrors可以配置多个mirror，每个mirror有id,name,url,mirrorOf属性，id是唯一标识一个mirror就不多说了，name貌似没多大用，相当于描述，url是官方的库地址，mirrorOf代表了一个镜像的替代位置，例如central就表示代替官方的中央库。
> 2. 镜像库并不是一个分库的概念，就是说当a.jar在第一个mirror中不存在的时候，maven会去第二个mirror中查询下载。但事实却不是这样，当第一个mirror中不存在a.jar的时候，并不会去第二个mirror中查找，甚至于，maven根本不会去其他的mirror地址查询。
> 3. maven的mirror是镜像，而不是“分库”，只有当前一个mirror无法连接的时候，才会去找后一个，类似于备份和容灾。
> 4. mirror也不是按settings.xml中写的那样的顺序来查询的。所谓的第一个并不一定是最上面的那个。当有id为B,A,C的顺序的mirror在mirrors节点中，maven会根据字母排序来指定第一个，所以不管怎么排列，一定会找到A这个mirror来进行查找，当A无法连接，出现意外的情况下，才会去B查询。

# 7.Maven工程发布与私库
一般情况下，一个使用Maven的Java团队，都会有自己的项目仓库，当开发完成相应的功能之后，都会发布相应的jar/war包到私服。供其它项目成员使用。
1. 发布配置
在Maven工程的pom.xml中增加以下内容
```xml
<!-- 工程自述 -->
<groupId>org.aeonluck</groupId>
<artifactId>org.aeonluck.hello</artifactId>
<packaging>jar</packaging>
<version>0.1.0-SNAPSHOT</version>

<!-- 发布配置 -->
<distributionManagement>  
    <repository>  
        <id>snapshots</id>  
        <name>Snapshots</name>  
        <url>http://nexus.local.me/nexus/content/repositories/snapshots/</url>  
    </repository>  
</distributionManagement>  
```

> 重要提示：  
> 1. 当发布到属性为`snapshot`的仓库时，工程自述说明小节中的 `<version>`，需要带上后缀 `SNAPSHOT`，否则会当成 `release` 发布。会收到 `400 Bad Request` 错误。
> 2. 发布配置中的 `<url>` 地址一定要区分清楚。并在 `release` 与 `snapshot` 仓库里配置不同的地址。此外，**id**、**name** 要与Nexus中仓库的配置 **configuration** 中的内容一致。
> 3. Nexus OSS的仓库属性：
> - snapshot : 快照版本，即不稳定的频繁发布场所，常用于团队成员快速联调。
> - release : 正式版本，即稳定包所在场所。

2. 发布jar到的私库
```shell
$ cd /path/to/myproject
$ mvn deploy
```
包发布之后，我们即可在Nexus OSS的管理仓库中看到我们刚才发布的包。

**小提示** : 
```shell
$ mvn install:install-file -Dfile=jar包的位置 -DgroupId=上面的groupId -DartifactId=上面的artifactId -Dversion=上面的version -Dpackaging=jar

# 发布到本地仓库
mvn install:install-file -DgroupId=com.bea.xml -DartifactId=jsr173-ri -Dversion=1.0 -Dpackaging=jar -Dfile=[path to file]

# 发布到私服
mvn deploy:deploy-file -DgroupId=com.bea.xml -DartifactId=jsr173-ri -Dversion=1.0 -Dpackaging=jar -Dfile=[path to file] -Durl=[url] -DrepositoryId=[id]
```
> 重要参考：
> 1. [关于Maven deploy](https://maven.apache.org/plugins/maven-deploy-plugin/deploy-file-mojo.html)
> 2. Maven默认本地仓库位于`~/.m2/repository`，若在pom中未指定发布地址，则默认发布到此处。此配置可在Maven的 **settings.xml** 修改，即修改 **localRepository**，注意，只能是路 **文件路径**，不能是远程url。


# 5.附录
- [Building Java Projects with Maven](https://spring.io/guides/gs/maven/)
