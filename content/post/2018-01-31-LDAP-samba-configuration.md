---
title: Ubuntu 16.04 Samba 使用 openldap 验证
date: 2018-01-31T10:53:41+08:00
tags:
---

## Directory Services(目录服务)

我们知道，当局域网的规模变的越来越大时，为了方便主机管理，我们使用DHCP来实现IP地址、以太网地址、主机名和拓扑结构等的集中管理和统一分配。同样，如果一个局域网内有许多的其它资源时，如打印机、共享文件夹等等，为了方便的定位及查找它们，一种集中定位管理的方式或许是较好的选择，DNS和NIS都是用来实现类似管理的方法。

对于局域网内的一个用户来讲，工作等其它应用需要，我们必须凭帐号登录主机、用帐号收发E-mail，甚至为了管理需要公司还需要维护一个电子号码簿来存储员工的姓名、地址、电话号码等信息。随着时间的增长，我们会为这些越来越多的帐号和密码弄的头晕脑胀。同时，如果一个员工离开，管理员就不得不翻遍所有的记录帐号信息的文件把离职员工的信息删除。这些将是一个繁琐而效率低下的工作。那么，如果能将此些帐号信息等统一到一个文件中进行管理，无疑会大大提高员工及管理员的工作效率。目录服务（LDAP是其实现的一种）正是基于这些应用实现的。

## LDAP

LDAP是Lightweight Directory Access Protocol的缩写，顾名思义，它是指轻量级目录访问协议（这个主要是相对另一目录访问协议X.500而言的；LDAP略去了x.500中许多不太常用的功能，且以TCP/IP协议为基础）。目录服务和数据库很类似，但又有着很大的不同之处。数据库设计为方便读写，但目录服务专门进行了读优化的设计，因此不太适合于经常有写操作的数据存储。同时，LDAP只是一个协议，它没有涉及到如何存储这些信息，因此还需要一个后端数据库组件来实现。这些后端可以 是bdb(BerkeleyDB)、ldbm、shell和passwd等。

LDAP目录以树状的层次结构来存储数据（这很类同于DNS），最顶层即根部称作“基准DN”，形如"dc=mydomain,dc=org"或者"o= mydomain.org"，前一种方式更为灵活也是Windows AD中使用的方式。在根目录的下面有很多的文件和目录，为了把这些大量的数据从逻辑上分开，LDAP像其它的目录服务协议一样使用OU （Organization Unit），可以用来表示公司内部机构，如部门等，也可以用来表示设备、人员等。同时OU还可以有子OU，用来表示更为细致的分类。

LDAP中每一条记录都有一个唯一的区别于其它记录的名字DN（Distinguished Name）,其处在“叶子”位置的部分称作RDN；如dn:cn=tom,ou=animals,dc=mydomain,dc=org中tom即为 RDN；RDN在一个OU中必须是唯一的。

因为LDAP数据是“树”状的，而且这棵树是可以无限延伸的，假设你要树上的一个苹果（一条记录），你怎么告诉园丁它的位置呢？当然首先要说明是哪一棵树（dc，相当于MYSQL的DB），然后是从树根到那个苹果所经过的所有“分叉”（ou，呵呵MYSQL里面好象没有这 DD），最后就是这个苹果的名字（uid，记得我们设计MYSQL或其它数据库表时，通常为了方便管理而加上一个‘id’字段吗？）。好了！这时我们可以清晰的指明这个苹果的位置了，就是那棵“歪脖树”的东边那个分叉上的靠西边那个分叉的再靠北边的分叉上的半红半绿的……，晕了！你直接爬上去吧！我还是说说LDAP里要怎么定义一个字段的位置吧，树（dc=waibo,dc=com)，分叉（ou=bei,ou=xi,ou= dong），苹果（cn=honglv），好了！位置出来了：
```
dn:cn=honglv,ou=bei,ou=xi,ou=dong,dc=waibo,dc=com
```

## Samba
Samba是由Andrew Tridgell在1991年（和Linux诞生的时间接近）制作的，当时他使用的是DEC的Pathworks网络，但是他发现无法同时使用Sun的NFS协议（正如我们前面介绍的，NFS是一个非常有用的网络协议），于是，连Socket（套接字）都不熟悉的他开始尝试自己在PC机上实现NFS，经过不断的摸索，他终于在自己的计算机上实现了NFS，采用的网络协议是NetBIOS（因为NetBIOS是公开的，可以合法地得到）。到了1992年1月，他开发出了0.1版，称为Server 0.1，随后又开发了一段时间，由于得到了X终端，他放弃了进一步的开发。直到1992年底，从一封电子邮件中，Andrew Tridgell获知了Linux，一个爱好者将Server 1.0转换到了Linux上，很快，人们发现这个程序可以直接使用，应用户的要求，Adrew Tridgell开始在Linux上开发，同时他发现smb-server已经被别人注册了，所以就只好起名为Samba，这就是Samba这个名称的由来。

Samba是种自由软件，用来让UNIX系列的操作系统与微软Windows操作系统的SMB/CIFS(Server Message Block/Common Internet File System)网络协定做连结。不仅可存取及分享SMB的资料夹及打印机，本身还可以整合入Windows Server的网域、扮演为网域控制站(Domain Controller)以及加入Active Directory成员。简而言之，此软件在Windows与UNIX系列OS之间搭起一座桥梁，让两者的资源可互通有无。

## 集成安装
以下安装基于Ubuntu 16.04 docker

### Samba
首先安装和配置Samba，可以通过apt来安装
```
apt-get install -y samba samba-common python-glade2 system-config-samba
```
备份配置文件，生成一个新的空的配置文件。
```
cp -pf /etc/samba/smb.conf /etc/samba/smb.conf.bak
cat /dev/null  > /etc/samba/smb.conf
```
添加以下内容到配置文件，生成一个匿名共享目录。
```
[global]
workgroup = WORKGROUP
server string = Samba Server %v
netbios name = ubuntu
security = user
map to guest = bad user
dns proxy = no

#============================ Share Definitions ==============================

[Anonymous]
path = /samba/anonymous
browsable =yes
writable = yes
guest ok = yes
read only = no
```
生成共享目录,修改权限。
```
mkdir -p /samba/anonymous
chmod -R 0775 /samba/anonymous
chown -R nobody:nogroup /samba/anonymous
```
重启服务
```
service smbd restart
```
可以测试一下匿名共享目录是否可以访问，windows("\\ubuntu\Anonymous")或者Linux(smbclient //ubuntu/Anonymous)。如果有问题就需要查看log检查问题。


### Openldap
用apt安装openldap
```
sudo apt install slapd ldap-utils
```
如果需要修改配置可以用以下命令重新配置
```
sudo dpkg-reconfigure slapd
```
- Omit OpenLDAP server configuration—answer No
- DNS domain name—enter your correct A record for your domain name (or a subdomain)
- Organization Name—enter the name of your organization (such as company or division)
- Administrator password—enter the same password you used during the installation
- Database backend—select MDB
- Remove database—select No
- Move old database—select Yes

如果不太习惯命令行可以使用Web界面，安装phpldapadmin。
```
sudo apt install phpldapadmin
```
安装完成之后编辑/etc/phpldapadmin/config.php修改配置。如果没有修改base DN是可以直接使用的，修改了就做对应的修改就好了。可以打开浏览器访问 http://ubuntu/phpldapadmin 测试一下。

关于openldap和samba的更多配置可以查看参考连接。

### Samba and LDAP
首先安装两个软件
```
sudo apt install samba smbldap-tools
```

### LDAP 配置
- Import a schema
- Index some entries
- Add objects

用以下命令导入samba的schema
```
zcat /usr/share/doc/samba/examples/LDAP/samba.ldif.gz | sudo ldapadd -Q -Y EXTERNAL -H ldapi:///
```
检查一下导入的schema
```
sudo ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b cn=schema,cn=config 'cn=*samba*'
```
创建一个samba_indices.ldif文件，导入一下内容
```
dn: olcDatabase={1}mdb,cn=config
changetype: modify
replace: olcDbIndex
olcDbIndex: objectClass eq
olcDbIndex: uidNumber,gidNumber eq
olcDbIndex: loginShell eq
olcDbIndex: uid,cn eq,sub
olcDbIndex: memberUid eq,sub
olcDbIndex: member,uniqueMember eq
olcDbIndex: sambaSID eq
olcDbIndex: sambaPrimaryGroupSID eq
olcDbIndex: sambaGroupType eq
olcDbIndex: sambaSIDList eq
olcDbIndex: sambaDomainName eq
olcDbIndex: default sub,eq
```
加载新的文件
```
sudo ldapmodify -Q -Y EXTERNAL -H ldapi:/// -f samba_indices.ldif
```
检查一下
```
sudo ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b cn=config olcDatabase={1}mdb olcDbIndex
```
添加samba ldap objects
```
sudo smbldap-config
```
这里需要特别注意，我在这里配置了很多次才成功，需要特别设置的值，workgroup name, ldap suffix, ldap master bind dn。注意，如果没有slave的话，需要在配置文件中注释掉，否则服务会起不来的。具体的配置可以参考 https://www.unixmen.com/setup-samba-domain-controller-with-openldap-backend-in-ubuntu-13-04/ 。

备份一下
```
sudo slapcat -l backup.ldif
```
导入本地用户
```
sudo smbldap-populate -g 10000 -u 10000 -r 10000
```
如果没有使用本地数据库的话，这一步会失败，可以重配来解决这个问题。

### Samba 配置
修改配置文件，把一下内容添加到配置文件, 如果没有使用ssl，设置ldap ssl = off。相关的值替换为你自己的值。像user，group在smbldap-populate执行后会生成。
```
#  passdb backend = tdbsam
   workgroup = EXAMPLE

# LDAP Settings
   passdb backend = ldapsam:ldap://hostname
   ldap suffix = dc=example,dc=com
   ldap user suffix = ou=People
   ldap group suffix = ou=Groups
   ldap machine suffix = ou=Computers
   ldap idmap suffix = ou=Idmap
   ldap admin dn = cn=admin,dc=example,dc=com
   # or off if TLS/SSL is not configured
   ldap ssl = start tls
   ldap passwd sync = yes
```
修改完成后使用testparm -s测试一下配置文件是否存在错误。
配置完成后添加一个用户就可以测试环境是否存在问题。


link:
- http://czmmiao.iteye.com/blog/1561597
- https://www.unixmen.com/setup-samba-domain-controller-with-openldap-backend-in-ubuntu-13-04/
- https://www.howtoforge.com/tutorial/samba-server-ubuntu-16-04/
- https://help.ubuntu.com/lts/serverguide/openldap-server.html
