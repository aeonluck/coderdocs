# 1.前言
## 1.1.编写目的
git作为取代CVS，SVN的新一代源码版本管理系统，从诞生之初，就引起了巨大的反响。随着git的不断推广与成熟，基于git相关的github更是开启了开源界的一片大好。gitlab作为与github相似的开源免费管理系统，为我们开发提供了更好的工具，极大了便利了软件开发团队，提升了开发与协作效率。
本文就gitlab的在Cent OS 7下的安装与配置作简要的说明。

# 2.内容
## 2.1.安装前的准备
1. 安装yum
对于RedHat Linux系的发行版来说，YUM是我们常用的包管理系统之一。yum已经集成在大多数版本中。关于yum的安装这里不在累述。

2. 安装基本依赖
```shell
$ sudo yum install curl policycoreutils openssh-server openssh-clients
$ sudo systemctl enable sshd
$ sudo systemctl start sshd
$ sudo yum install postfix
$ sudo systemctl enable postfix
$ sudo systemctl start postfix
$ sudo firewall-cmd --permanent --add-service=http
$ sudo systemctl reload firewalld
```

中国大陆用户，可以采用清华大学的软件源进行安装，方法如下：
1. 在YUM中增加清华大学软件源。
```shell
# cd /etc/yum.repo.d
# vim gitlab-ce.repo

内容如下：
[gitlab-ce]
name=gitlab-ce
baseurl=http://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7
repo_gpgcheck=0
gpgcheck=0
enabled=1
gpgkey=https://packages.gitlab.com/gpg.key
```
保存退出。

2. 更新yum
```shell
# yum makecache
```

## 2.2.执行安装
1. 安装gitlab主程序：gitlab-c【失效】
```shell
$ curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
$ sudo yum install gitlab-ce
``` 
如果习惯RPM方式安装，方法如下：
```shell
# curl -LJO https://packages.gitlab.com/gitlab/gitlab-ce/packages/el/7/gitlab-ce-XXX.rpm/download
# rpm -i gitlab-ce-XXX.rpm
```
> 官方下载地址：https://about.gitlab.com/downloads/archives/
> PS : 藏得尼玛鬼深 T_T

2. 清华大学软件源安装【已关闭，失效】
```shell
# yum install gitlab-ce
```

## 2.3.配置
### 2.3.1.通用配置
1. 默认配置
默认情况下，gitlab通过omnibus package包已经内置了nginx, redis, unicorn等必要的组件。并且默认占用80端口。因此，如果你不需要其它变动，可以直接进行使用。
浏览器打开`http://localhost`即可进入到gitlab初始root帐户密码设置页面。

2. 设定域名或端口
```shell
# vim /etc/gitalb/gitlab.rb

" 第 11 行，修改为你的gitlab域名
external_url 'http://gitlab.aeonluck.me'

" 607 行，修改内置NGINX使用的端口80为其它端口，如：8081
nginx['listen_port'] = 8081
```

3. 使用非绑定webserver
- 配置外部webserver运行用户
一般情况下，你可能已经安装了nginx/apache类似的webserver，如果你想使用自定义的webserver，gitlab提供了相应的方法。此处以nginx为例：
```shell
# vim /etc/gitalb/gitlab.rb

" 578 行，设定自定义webserver运行用户
" 注意！此处用户应与你的nginx.conf中的 `use xxxx` 语句配置相同。
web_server['external_users'] = ['www']

" 591 行，关闭内置nginx
 nginx['enable'] = false
```

> 注意：  
> gitlab非内置nginx时，默认使用8080端口。因此需要注意端口与别的应用是否存在冲突。

- 将www加入gitlab-www组，否则外部nginx将无法访问gitlab运行目录。
```
usermod -aG gitlab-www www
```

> 关于 ***gitlab 502 错误的解决方法：**：  
> 如果不修改用户组，即使`chmod -R 777 /var/opt/gitlab`也是无效的。
> 你会收到 ***502*** 错误。

- 在外部nginx中添加反向代理：
```
# vim /usr/local/nginx/conf/vhosts/gitlab.conf

" 内容如下：
upstream gitlab-workhorse {
  server unix://var/opt/gitlab/gitlab-workhorse/socket fail_timeout=0;
}

server {
  server_name gitlab.your-domain.com;
  server_tokens off;
  root /opt/gitlab/embedded/service/gitlab-rails/public;

  client_max_body_size 250m;

  access_log  /var/log/nginx/gitlab.your-domain.com/access.log;
  error_log   /var/log/nginx/gitlab.your-domain.com/error.log;

  location / {
    proxy_read_timeout      3600;
    proxy_connect_timeout   300;
    proxy_redirect          off;
    proxy_buffering off;

    proxy_set_header    Host                $http_host;
    proxy_set_header    X-Real-IP           $remote_addr;
    proxy_set_header    X-Forwarded-For     $proxy_add_x_forwarded_for;
    proxy_set_header    X-Forwarded-Proto   $scheme;

    proxy_pass http://gitlab-workhorse;
  }

  error_page 502 /502.html;
}

" 重启nginx使配置生效
# nginx -s reload
```

### 2.3.2.设定邮件服务器
```shell
# vim /etc/gitlab/gitlab.rb

" 334行，配置smtp服务器
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.xxx.com"
gitlab_rails['smtp_port'] = 25
gitlab_rails['smtp_user_name'] = "yourname@xxx.com"
gitlab_rails['smtp_password'] = "your_password"
gitlab_rails['smtp_domain'] = "xxx.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = false

```

### 2.3.3.开启LDAP认证
```shell
# vim /etc/gitlab/gitlab.rb

" 108 行，开启ldap认证
gitlab_rails['ldap_enabled'] = true
gitlab_rails['ldap_servers'] = YAML.load <<-'EOS' # remember to close this block with 'EOS' below
main: # 'main' is the GitLab 'provider ID' of this LDAP server
     label: 'LDAP'
     enabled: true
     host: 'localhost'
     port: 389
     uid: 'cn'
     method: 'plain' # "tls" or "ssl" or "plain"
     bind_dn: 'CN=Manager,DC=aeonluck,DC=me'
     password: '{SSHA}wbyH7h7HPqCzvOBH4L73ZnZUX+hfPI3T'
     active_directory: true
     allow_username_or_email_login: false 
     block_auto_created_users: false
     base: 'DC=aeonluck,DC=me'
     user_filter: ''
     attributes:
       username: ['cn']
       email:    ['mail', 'email', 'userPrincipalName']
       name:       'cn'
       first_name: 'givenName'
       last_name:  'sn'
     ## EE only
     group_base: ''
     admin_group: ''
     sync_ssh_keys: false

" 158 行，去掉#，闭合EOS
EOS
```

4. 更新gitlab配置
```shell
# gitlab-ctl reconfigure
```

5. 启动与关闭gitlab
```shell
gitlab-ctl start
gitlab-ctl stop
gitlab-ctl restart
```

# 3.附录
1. [gitlab官网](https://about.gitlab.com/downloads/)
2. [gitlab清华大学源](https://mirror.tuna.tsinghua.edu.cn/help/gitlab-ce/)   
3. [gitlab相关配置](https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/doc/settings/nginx.md#setting-the-nginx-listen-port)
