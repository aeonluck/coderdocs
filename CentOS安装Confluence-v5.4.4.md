# 1.前言
Atlanssian Confluence作为一款主流的多人文档协作软件，在各大软件开发团队都有成功应用，大大地帮助开发及产品人员提高了协作效率。本文就atlanssian confluence 5.x的安装与破解作出相应说明（本文以Linux下的安装为例）。


# 2.内容
## 2.1.安装前的准备
1. Confluece支持自持或是外挂到tomcat两种运行方式（自持方式其实内嵌了tomcat），本文仅以自持方式为说明场景。
下载oracle jdk rpm包：http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html，执行安装：
```shell
# cd /path/do/downloads
# rpm -ivh jdk-8u74-linux-x64.rpm
```

2. 安装mysql server（注意，必须是5.6及以下版本）
```shell
# yum install mysql mysql-server
```

> 注意！  
> - Confluence不支持oracle mysql v5.7.*版本，请下载mysql v5.5.* ~ mysql v5.6.*对应版本。
> - 推荐安装mariaDB(https://mariadb.org)，实测没有发现兼容问题。

3. 下载Confluence安装包：http://pan.baidu.com/s/1jGVRn5G

## 2.2.安装confluence与破解
1. 解压得到的安装包，执行安装。
```shell
# cd /path/to/downloads
# tar confluence-5.4.4.tar ./
# cd ./confluence-5.4.4
# chmod +x ./atlassian-confluence-5.4.4-x64.bin
# ./atlassian-confluence-5.4.4-x64.bin
```
> 注意！  
> - 在安装时，建议选择自定义安装，以便可以修改安装目录，数据目录与端口，以及是否作为独立服务。
> - 本文以独立服务为实现方式。关于嵌入其它的应用服务器如tomcat的方法，请查询相关资料。

2.安装confluence mysql连接器驱动，复制安装包中的mysql驱动到confluence安装目录下的lib目录下，重启confluence服务。
```shell
# cd /path/to/confluence-5.4.4/lib
# cp mysql-connector-java-5.1.26-bin.jar ./
```

3. 破解！复制安装包中的“atlassian-extras-2.4.jar”到安装目录下的confluence/WEB-INF/lib目录下，重启confluence服务即完成相关破解。
```shell
# cd /usr/local/confluence-5.4.4/confluence/WEB-INF/lib
# cp /path/to/downlodas/atlassian-extras-2.4.jar ./

# service confluence restart
```

## 2.3.配置confluence
1. 打开 http://localhost:8090, 按提示申请试用key（需要打开atlanssian的官网，进行注册，按提示填写资料后生成一个试用key。)，得到试用key后，复制填写到对应的licence key输入框中提交，提交进入下一步。

2. 选择“产品模式”（production mode)。

3. 回到页面，点击下一步，配置数据库，Confluence支持内嵌数据库或外置数据库存储两种方式。如果你需要独立的外部数据库存储，那么选择"External Database"，本文以Mysql为主，填写数据库连接资料（用户名，密码）。特别要注意的是，要在连接字串位置最后加入utf8+innodb支持（相关字符串在页面上有文字说明，注意阅读）。
```shell
mysql> CREATE DATABASE IF NOT EXISTS confluence default charset utf8 COLLATE utf8_general_ci;
mysql> grant all privileges on confluence.* to confluence@localhost identified by 'password'; 
mysql> flush privileges; 
```

> 注意！  
> 为了支持中文存储，必须预先在mysql中修改所有连接字符集为utf8，新建一个名为confluence且字符集为utf8的数据库。**否则将不存储中文，在提交中文信息时程序会报错！**

4. 数据库配置完成后，进入下一步完成安装。如果你是第一次用，可以选择生成一个“样例网站”，如果不需要，则选择空站点。


## 2.4.中文化配置
1. 登录后台，打开右上角“设置”图标下的“Aadd ons"安装语言包扩展。上传安装包中的中文包：Confluence-5.4.4-language-pack-zh_CN.jar，然后在左侧”Language"选择语言为中文即可。
2. 导出PDF的中文化支持。选择“PDF Export Language Support”，安装中文字体(TTF,TTC)格式，推荐“文泉驿微米黑”字体：http://wenq.org/wqy2/index.cgi

## 2.5.配置邮件服务器
后台 > 左侧管理菜单“邮件服务器”，填写相关smtp邮件服务器资料提交即可。


## 2.6.与OpenLDAP集成
**后台** > **用户目录** > **添加目录** > **LDAP(注意，不是内部的LDAP。)**，进入详细配置。
- 服务器设置部份：目录类型选择“OpenLDAP”，帐号填写你的LDAP管理员信息与密码，如：cn=manager,dc=mydomain,dc=com。
- LDAP模式部份：只需要填写你的LDAP基础域名部份，如：dc=mydomain,dc=com，附加用户DN与附加用户组DN不要填写，否则会出错。
- LDAP权限：选读写（Read/Write）模式。
- 其余部份不用填写，保持默认即可，点击“测试并提交”，如无意外将会显示“测试通过”类似意义字样。你也可以在下一次输入一个OpenLDAP中存在的用户进行进一步测试。

> 注意！  
> Confluence中无法创建属于LDAP中的组或是用户，但是用户密码修改会同步到LDAP中。如果需要管理LDAP中的用户或组，推荐安装JIRA或者phpldapadmin（php7不支持）。


# 3.番外篇
1. 如果需要重复安装，最好先做好数据备份，即备份安装时的confluence data目录。然后进入confluence安装目录进行御载。即：
```shell
# cd /path/to/confluence-5.4.4
# ./uninstall
```

**不然会confluence服务名为confluence1, confluence2...的形式递增。**删除服务：
```shell
# chkconfig --del service-name
```

启动confluence，用浏览器打开http://localhost:8090，进入confluence配置。
```shell
" 启动Confluence
service confluence start

; 重启confluence
service confluence restart
```

2. 如果你想对访问时外屏蔽端口8090，可以使用nginx反向代理实现，即:
```shell
# vim /etc/nginx/conf/vhosts/wiki.mydomain.com.conf

" 增加内容如下：
server｛
   server_name wiki.mydomain.com;
   location / {
        proxy_pass   http://127.0.0.1:8090;
        ; 传递真实的头部信息
        proxy_set_header X-Real-IP $remote_addr;
    }
}

" 重启nginx，即可使用不带端口的域名方式访问confluence，即：http://wiki.mydomain.com。
# service nginx reload

" 自定义安装采用如下方式
# /usr/local/nginx-1.9.12/sbin/nginx -s reload
```

如果您在安装或破解的过程中遇到以下错误：
- **Hibernate operation: Could not save object; uncategorized SQLException for SQL []; SQL state [HY000]; error code [1665];**
- **Non Clustered Confluence : Database is being updated by another Confluence instance. Please see ...**

解决方法如下：
```shell
# vim /etc/my.cnf

" 增加以下行
binlog_format=ROW
```

具体参见：http://shitouququ.blog.51cto.com/24569/1258266


# 4.备份还原
- 备份： Confluence默认会每日进行备份。备份文件以ZIP格式存放在数据目录（非安装目录）下的backup目录中。
- 还原：登入系统后点，找到“管理 > 备份与还原（restore)”，还原有多种方式：
1. 上传zip格式进行还原，此种情况适合备份包比较小的情况，一般200M左右。
2. 大文件备份还原，将zip文件上传至confluence运行目录（数据目录）下的restore，在管理后台“管理 > 备份与还原（restore) ” 选择“从Confluence主目录中由备份恢复”，在文件列表中点击选择需要还原的备份文件，点击“还原”即可，这需要耗费一段时间。**这里特别要注意的是：Confluence默认安装完堆内存为512m，如果备份包大小超过这个数，请先修改confluence的JVM运行时参数！切切！！！**具体操作如下：
```shell
" 进入confluence安装目录
# cd /path/to/$confluence_home/bin

# vim catalina.sh
" 在88行“cygwin=false”前加入：
JAVA_OPTS="-Xms1024m -Xmm1024m -XX:+UseGCOverheadLimit"

" 保存退出，重启confluence
# service confluence restart
```
> 注意！上面的例了中把堆大小设成了“1024M”，实际情况根据自己的服务器来决定。

> 小提示：日志查看：在$confluence_data_dir / log中。


# 5.结论
- 如果你觉得confluence确实是一款不错的产品，请您购买正版。
- 本文只提供一种工具方法，对产生的相关法律问题不担负任何责任。
- Confluence中文教程：http://www.confluence.cn/pages/viewpage.action?pageId=2916470
