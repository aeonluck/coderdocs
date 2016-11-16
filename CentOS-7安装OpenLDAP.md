# 1.前言
## 1.1.编写目的
CentOS 7上安装OpenLDAP的方法与CentOS 6.*上有所不同。centos 6.8上的slapd.conf已经被弃用。新的安装方式官方说明又跟实践中不太相符，按官方的操作总是不成功。
因此，为了让大家少踩坑，这里把经过验证的方法写给大家。


# 2.内容
1. 安装LDAP
```shell
[root@dlp ~]# yum -y install openldap-servers openldap-clients
[root@dlp ~]# cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG 
[root@dlp ~]# chown ldap. /var/lib/ldap/DB_CONFIG 
[root@dlp ~]# systemctl start slapd 
[root@dlp ~]# systemctl enable slapd 
```

2. 修改LDAP服务器管理密码
```shell
# 生成密码
[root@dlp ~]# slappasswd 
New password:
Re-enter new password:
{SSHA}xxxxxxxxxxxxxxxxxxxxxxxx

# 密码配置
[root@dlp ~]# vi chrootpw.ldif
# specify the password generated above for "olcRootPW" section
 dn: olcDatabase={0}config,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}xxxxxxxxxxxxxxxxxxxxxxxx

# 注入密码配置
[root@dlp ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f chrootpw.ldif 
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
modifying entry "olcDatabase={0}config,cn=config"
```

3. 注入数据结构
```shell
[root@dlp ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif 
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "cn=cosine,cn=schema,cn=config"

[root@dlp ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif 
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "cn=nis,cn=schema,cn=config"

[root@dlp ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif 
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "cn=inetorgperson,cn=schema,cn=config"
```

4. 自定义LDAP服务器
```shell
# 生成管理员密码
[root@dlp ~]# slappasswd 
New password:
Re-enter new password:
{SSHA}xxxxxxxxxxxxxxxxxxxxxxxx

# 自定义LDAP配置
[root@dlp ~]# vi chdomain.ldif
# 替换"dc=***,dc=***"为你的域名
# 替换"olcRootPW"为上面生成的密码
 dn: olcDatabase={1}monitor,cn=config
changetype: modify
replace: olcAccess
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth"
  read by dn.base="cn=Manager,dc=srv,dc=world" read by * none

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=srv,dc=world

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=Manager,dc=srv,dc=world

dn: olcDatabase={2}hdb,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}xxxxxxxxxxxxxxxxxxxxxxxx

dn: olcDatabase={2}hdb,cn=config
changetype: modify
add: olcAccess
olcAccess: {0}to attrs=userPassword,shadowLastChange by
  dn="cn=Manager,dc=srv,dc=world" write by anonymous auth by self write by * none
olcAccess: {1}to dn.base="" by * read
olcAccess: {2}to * by dn="cn=Manager,dc=srv,dc=world" write by * read

# 注入自定义LDAP配置
[root@dlp ~]# ldapmodify -Y EXTERNAL -H ldapi:/// -f chdomain.ldif 
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
modifying entry "olcDatabase={1}monitor,cn=config"

modifying entry "olcDatabase={2}hdb,cn=config"

modifying entry "olcDatabase={2}hdb,cn=config"

modifying entry "olcDatabase={2}hdb,cn=config"

# 定义数据节点
[root@dlp ~]# vi basedomain.ldif
# 替换"dc=***,dc=***"为你的域名
 dn: dc=srv,dc=world
objectClass: top
objectClass: dcObject
objectclass: organization
o: Server World
dc: Srv

dn: cn=Manager,dc=srv,dc=world
objectClass: organizationalRole
cn: Manager
description: Directory Manager

dn: ou=People,dc=srv,dc=world
objectClass: organizationalUnit
ou: People

dn: ou=Group,dc=srv,dc=world
objectClass: organizationalUnit
ou: Group

# 注入数据定义文件
[root@dlp ~]# ldapadd -x -D cn=Manager,dc=srv,dc=world -W -f basedomain.ldif 
Enter LDAP Password: # directory manager's password
adding new entry "dc=srv,dc=world"

adding new entry "cn=Manager,dc=srv,dc=world"

adding new entry "ou=People,dc=srv,dc=world"

adding new entry "ou=Group,dc=srv,dc=world"
```

5. 开放防火墙端口
```shell
[root@dlp ~]# firewall-cmd --add-service=ldap --permanent 
success
[root@dlp ~]# firewall-cmd --reload 
success
```

# 3.附录
- [Quick-Start Guide](http://www.openldap.org/doc/admin24/quickstart.html)
- [CentOS 7- Configure LDAP Server](http://www.server-world.info/en/note?os=CentOS_7&p=openldap)
