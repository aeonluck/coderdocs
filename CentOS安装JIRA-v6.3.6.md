# 1.前言
JIRA是Atlassian公司出品的项目与事务跟踪工具，被广泛应用于缺陷跟踪、客户服务、需求收集、流程审批、任务跟踪、项目跟踪和敏捷管理等工作领域。本文就JIRA v6.3.6的安装作具体说明。

# 2.内容
## 2.1.安装前的准备
- JDK ： JIRA运行环境，参见：http://blog.163.com/sujoe_2006/blog/static/3353151201622253331523/
- MySQL：数据库存储，参见：http://blog.163.com/sujoe_2006/blog/static/33531512016223111231266/
- 下载JIRA安装包：http://pan.baidu.com/s/1o7nuzME


## 2.2.正式安装JIRA
安装十分简单，直接解压JIRA安装包。
```shell
# cd /path/to/downloads
# tar -xzvf atlassian-jira-6.3.6.tar.gz ./
# mv atlassian-jira-6.3.6-standalone /usr/local/jira-6.3.6

; 建立jira对外运行目录
# mkdir /webapps /webapps/jira.mydomain.com
```

修改JIRA配置，指定jira_home，即：
```shell
# vim /usr/local/jira-6.3.6/atlassian-jira/WEB-INF/classes/jira-application.properties 

jira.home = /webapps/jira.mydomain.com
```

启动jira服务，即：
```shell
# /usr/local/jira-6.3.6/bin/start-jira.sh
```
输出如下内容：
```
To run JIRA in the foreground, start the server with start-jira.sh -fg
……
Tomcat started.
```

至此，JIRA的基本安装结束。接下来我们要安装一下JIRA需要用到的MySQL驱动包，
```shell
# cd /path/to/downloads
# cp mysql-connector-java-5.1.7-bin.jar /usr/local/jira-6.3.6/atlassian-jira/WEB-INF/lib
```

重启JIRA
```shell
# killall -TERM jira
# /usr/local/jira-6.3.6/bin/start-jira.sh
```

配置JIRA数据库连接：
```shell
mysql> CREATE DATABASE `jira` CHARACTER SET utf8 COLLATE utf8_general_ci;
mysql> GRANT ALL ON jira.* TO jira_user@'localhost' IDENTIFIED BY 'my password';
```

继续按页面填写相关站点信息，域名等。然后选择自定义的JIRA运作模式（项目管理，软件开发，IT服务桌面），通常选择“项目管理”模式，下一步，选择“I have a JIRA key”，填写以下临时授权key：
`AAABBw0ODAoPeNptkFtLxDAQhd/zKwI+R9Kwy66FPKxthGhvtF0p4kuso0a6sUwvuP/edissyj4MDHPOfHOYqzu0tICWeoJy4a+FzzkNwpIK7q1ICF2Ntu3tl5P3Ot89+1SNphnMPCEBwqkJTQ9y9jN+wzxBPi2a68jW4DpQr/a0rZJS5VmuC0XOBNnjAH/s5bGFxBxABmkcqzzQu2jRTd3bEZaFZvE+AnYzRJDYWNeDM64G9d1aPJ4TeXxOlOK7cbZbjrbNgkyGwwtg+rbvJpBkHikAR0Adytt0XzFV7R5Y+qQzVkWZIoVK5FQsWq03YrvdkN/Ekz3S4SXlcpRswPrDdPD/aT+P1nzDMC0CFQCM9+0LlHVNnZQnSTwuRO3eK+2gVgIUCteTs4Q3khIgrnsY64hxYB/d8bM=X02dh
`

提交注册后，填写管理员相关信息（用户名，邮箱，密码），然后进入一步“邮件服务器设置”，此步也可以选择跳过，安装完成后通过管理后台进行设置即可。之后完成JIRA的全部安装过程，页面重定向到JIRA首页。


## 2.3.破解JIRA
```shell
# cd /path/to/downloads
# cp atlassian-extras-2.2.2.jar /usr/local/jira-6.3.6/atlassian-jira/WEB-INF/lib/
# cp atlassian-universal-plugin-manager-plugin-2.17.13.jar /usr/local/jira-6.3.6/atlassian-jira/WEB-INF/atlassian-bundled-plubins/

" 重启jira，进入破解：
# killall -TERM jira
# jira-6.3.6/bin/start-jira.sh
```

点击右上角齿轮形状的管理图标，选择“系统”，再选择“授权”，看到使用日期不到1个月。在进一步之前我们要完成以下工作：
1. 到atlassian官网申请一个可用的licence，地址：https://id.atlassian.com/login。
2. 登入之后，在此页面 https://my.atlassian.com/product申请一个“New Evaluation License”。"Server ID"填写我们在jira后台看到的ID，即：B7OT-4FTO-HTCL-TU0W。
3. 复制第2步中得到的KEY，替换以下内容中的"LicenseID"，SEN替换为生成KEY得到的SEN ID。授权码参数范例如下：
```
Description=JIRA: Commercial,
CreationDate=你的安装日期，格式（yyyy-mm-dd）,
jira.LicenseEdition=ENTERPRISE,
Evaluation=false,
jira.LicenseTypeName=COMMERCIAL,
jira.active=true,
licenseVersion=2,
MaintenanceExpiryDate=你想设置的失效日期如：2099-12-31,
Organisation=joiandjoin,
SEN=你申请到的SEN注意没有前缀LID,
ServerID=你申请到的ServerID,
jira.NumberOfUsers=-1,
LicenseID=LID你申请到的SEN，注意LID前缀不要丢掉,
LicenseExpiryDate=你想设置的失效日期如：2099-12-31,
PurchaseDate=你的安装日期，格式（yyyy-mm-dd）
```

示例如下（注意修改日期和ServerID, SEN）：
```
Description=JIRA: Commercial,
CreationDate=2015-08-17,
jira.LicenseEdition=ENTERPRISE,
Evaluation=false,
jira.LicenseTypeName=COMMERCIAL,
jira.active=true,
licenseVersion=2,
MaintenanceExpiryDate=2099-12-31,
Organisation=pl,
SEN=SEN-L4572887,
ServerID=B7OT-4FTO-HTCL-TU0W,
jira.NumberOfUsers=-1,
LicenseID=AAABBw0ODAoPeNptkFtLxDAQhd/zKwI+R9Kwy66FPKxthGhvtF0p4kuso0a6sUwvuP/edissyj4MDHPOfHOYqzu0tICWeoJy4a+FzzkNwpIK7q1ICF2Ntu3tl5P3Ot89+1SNphnMPCEBwqkJTQ9y9jN+wzxBPi2a68jW4DpQr/a0rZJS5VmuC0XOBNnjAH/s5bGFxBxABmkcqzzQu2jRTd3bEZaFZvE+AnYzRJDYWNeDM64G9d1aPJ4TeXxOlOK7cbZbjrbNgkyGwwtg+rbvJpBkHikAR0Adytt0XzFV7R5Y+qQzVkWZIoVK5FQsWq03YrvdkN/Ekz3S4SXlcpRswPrDdPD/aT+P1nzDMC0CFQCM9+0LlHVNnZQnSTwuRO3eK+2gVgIUCteTs4Q3khIgrnsY64hxYB/d8bM=X02dh,
LicenseExpiryDate=2099-12-31,
PurchaseDate=2015-08-17
```
将以上授权码信息填入授权码输入框，点击“增加“按钮”，页面刷新后，看到授权信息更新为“2099-12-31”，就表示破解成功。


## 2.4.汉化JIRA
使用安装时定义的管理员用户进入后台，点击左侧管理菜单“Add-ons”(插件管理)，上传安装包中的“JIRA-6.3.3-language-pack-zh_CN.jar”，上传成功能，页面将刷新为中文，或者手动设置站点语言为中文即可。


## 2.5.配置JIRA
### 2.5.1.反向代理访问JIRA
JIRA默认是使用带端口的方式提供对外访问的，页面中有大量的js中会定向到8080端口，如果你想对外屏蔽8080端口，并使用Nginx反向代理方式访问JIRA的话，则需要对jira相关的connector进行相应的配置，否则前台页面运行时会产生大量的js异常。

#### 2.5.1.1.配置nginx中反向代理到8080端口
```shell
server
{
   server_name jira.mydomain.com;

   location / {
        proxy_pass   http://127.0.0.1:8080;
          
        proxy_redirect  off; 
        proxy_set_header Host $host; 
        proxy_set_header X-Real-IP $remote_addr; 
        proxy_set_header X-Forwarded-For 
        $proxy_add_x_forwarded_for; 
   }
}
```

#### 2.5.1.2.修改jira服务器配置
```
# vim /usr/local/jira-6.3.6/jira/conf/server.xml

" 修改connector内容，增加proxyName, proxyPort配置：
<Connector 
acceptCount="100" 
connectionTimeout="20000" 
disableUploadTimeout="true" 
enableLookups="false" 
maxHttpHeaderSize="8192" 
maxThreads="150" 
minSpareThreads="25" 
port="8080" 
protocol="HTTP/1.1" 
redirectPort="8443" 
useBodyEncodingForURI="true" 
proxyName="jira.mydomain.com" 
proxyPort="80"
/> 
```

重启nginx与jira，即可使用不带端口的方式的方式访问Jira服务器，即：http://jira.mydomain.com。
```shell
# service nginx reload

" 重启jira
# killall -TERM jira
# /usr/local/jira-6.3.6/bin/start-jira.sh
```

#### 2.5.1.3.JIRA与OpenLDAP设置
JIRA可以与OpenLDAP相结合，实现用户、用户组管理，并可通过OpenLDAP进行同步。详情可参见：http://blog.163.com/sujoe_2006/blog/static/3353151201622253331523/


# 3.结论
1. jira与nginx设置参考：http://blog.servicerocket.com/adoption/blog/2014/07/3-steps-in-set-up-nginx-as-proxy-server-for-atlassian-jira
2. JIRA安装参考：http://blog.itpub.net/26230597/viewspace-1275597
