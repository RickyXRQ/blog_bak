title: Extmail for Linux CentOS 5
date: 2015-05-11 22:33:24
tags: Linux
---
## 配置环境
整个配置过程要求在root身份下进行，并且在配置过程中，如果没有特别的更换文件目录的情况，最好用 “cd ~”指令切换当前目录；
参考 http://wiki.extmail.org/extmail_solution_for_linux_centos-5 的extmail_solution_for_linux_cenos-5参考文档，并且采用的邮箱服务器为32bit的CentOS系统。且SELinux处于禁用状态。
## 具体过程
### 制作yum仓库
安装所需要的基础服务的rpm包，键入以下指令：
``` bash
$ yum install httpd mysql mysql-server mysql-devel openssl-devel dovecot perl-DBD-MySQL tcl tcl-devel libart_lgpl libart_lgpl-devel libtool-ltdl libtool-ltdl-devel expect
```
<!-- more -->
采用Extmail官方网站的yum源
``` bash
$ vi /etc/yum.repos.d/EMOS-Base.repo
```
加入以下内容：
``` bash
# EMOS-Base.repo
#
# Created by ExtMail Dev Team: http://www.extmail.org/
#
# $Id$

[EMOS-base]
name=EMOS-Base
baseurl=http://mirror.extmail.org/yum/emos/1.5/os/$basearch/
gpgcheck=0
priority=0
protect=0

[EMOS-update]
name=EMOS-Updates
baseurl=http://mirror.extmail.org/yum/emos/1.5/updates/$basearch/
gpgcheck=0
priority=0
protect=0
```
保存后，执行yum list等，确认成功。
### 配置MTA-Postfix
#### 安装postfix
``` bash
$ yum install postfix
$ rpm -e sendmail
```
#### 配置postfix
``` bash
$ postconf -n > /etc/postfix/main2.cf
$ mv /etc/postfix/main.cf /etc/postfix/main.cf.old
$ mv /etc/postfix/main2.cf /etc/postfix/main.cf
```
编辑mian.cf:
``` bash
$ vi /etc/postfix/main.cf
```
增加如下内容
``` bash
# hostname
mynetworks = 127.0.0.1
myhostname = mail.extmail.org
mydestination = $mynetworks $myhostname

# banner
mail_name = Postfix - by extmail.org
smtpd_banner = $myhostname ESMTP $mail_name

# response immediately
smtpd_error_sleep_time = 0s

# Message and return code control
message_size_limit = 5242880
mailbox_size_limit = 5242880
show_user_unknown_table_name = no

# Queue lifetime control
bounce_queue_lifetime = 1d
maximal_queue_lifetime = 1d
```
注意，此处的mydestination建议设置为空，即”mydestination = ”即可，否则在测试过程中会有冲突出现；
设置postfix为开机自启
``` bash
$ chkconfig postfix on
```
### 配置Courier-Authlib
#### 安装Courier-Authlib
安装以下软件包：
``` bash
$ yum install courier-authlib
$ yum install courier-authlib-mysql
```
编辑/etc/authlib/authmysqlrc文件
``` bash
$ vi /etc/authlib/authmysqlrc
```
将其内容清空，并且加入如下内容
``` bash
MYSQL_SERVER            localhost
MYSQL_USERNAME          extmail
MYSQL_PASSWORD          extmail
MYSQL_SOCKET            /var/lib/mysql/mysql.sock
MYSQL_PORT              3306
MYSQL_OPT               0
MYSQL_DATABASE          extmail
MYSQL_USER_TABLE        mailbox
MYSQL_CRYPT_PWFIELD     password
MYSQL_UID_FIELD         uidnumber
MYSQL_GID_FIELD         gidnumber
MYSQL_LOGIN_FIELD       username
MYSQL_HOME_FIELD        homedir
MYSQL_NAME_FIELD        name
MYSQL_MAILDIR_FIELD     maildir
MYSQL_QUOTA_FIELD       quota
MYSQL_SELECT_CLAUSE     SELECT username,password,"",uidnumber,gidnumber,\
                        CONCAT('/home/domains/',homedir),               \
                        CONCAT('/home/domains/',maildir),               \
                        quota,                                          \
                        name                                            \
                        FROM mailbox                                    \
                        WHERE username = '$(local_part)@$(domain)'
```
修改authdaemonrc文件
``` bash
$ vi /etc/authlib/authdaemonrc
```
修改如下内容：
``` bash
authmodulelist="authmysql"
authmodulelistorig="authmysql"
```
启动courier-authlib
``` bash
$ service courier-authlib start
```
如果返回如下信息，说明成功
``` bash
Starting Courier authentication services: authdaemond 
```
修改authdaemon socket目录权限
``` bash
$ chmod 755 /var/spool/authdaemon/
```
### 配置maildrop
#### 安装maildrop
``` bash
$yum install maildrop
```
#### 配置master.cf
为了使Postfix支持Maildrop，必须修改/etc/postfix/master.cf文件，注释掉原来的maildrop的配置内容，并改为：
``` bash
maildrop   unix        -       n        n        -        -        pipe
  flags=DRhu user=vuser argv=maildrop -w 90 -d ${user}@${nexthop} ${recipient} ${user} ${extension} {nexthop}
```
在main.cf中增加参数，使得maildrop支持多个收件人
``` bash
maildrop_destination_recipient_limit = 1
```
测试maildrop对authlib的支持
``` bash
$ maildrop -v
```
观察是否出现以下内容：
``` bash
maildrop 2.1.0 Copyright 1998-2005 Double Precision, Inc.
GDBM/DB extensions enabled.
Courier Authentication Library extension enabled.
Maildir quota extension enabled.
This program is distributed under the terms of the GNU General Public
License. See COPYING for additional information.
```
### 配置Apache
#### 虚拟主机设置
编辑httpd.conf文件
``` bash
$ vi /etc/httpd/conf/httpd.conf
```
在本文件的最后一行加上：
``` bash
NameVirtualHost *:80
Include conf/vhost_*.conf
```
编辑vhost_extmail.conf
``` bash
$ vi /etc/httpd/conf/vhost_extmail.conf
```
里面定义虚拟主机的相关内容：
``` bash
# VirtualHost for ExtMail Solution
<VirtualHost *:80>
ServerName mail.extmail.org
DocumentRoot /var/www/extsuite/extmail/html/

ScriptAlias /extmail/cgi/ /var/www/extsuite/extmail/cgi/
Alias /extmail /var/www/extsuite/extmail/html/

ScriptAlias /extman/cgi/ /var/www/extsuite/extman/cgi/
Alias /extman /var/www/extsuite/extman/html/

# Suexec config
SuexecUserGroup vuser vgroup
</VirtualHost>
```
设置apache为开机启动
``` bash
$ chkconfig httpd on
```
### 配置webmail-ExtMail
#### 安装ExtMail
``` bash
$ yum install extsuite-webmail
```
编辑webmail.cf
``` bash
$ cd /var/www/extsuite/extmail
$ cp webmail.cf.default webmail.cf
$ vi webmail.cf
```
主要变动的内容如下：
``` bash
SYS_MYSQL_USER = extmail
SYS_MYSQL_PASS = extmail
SYS_MYSQL_DB = extmail
```
更新cgi目录权限 由于SuEXEC的需要，必须将 extmailde的cgi目录修改成vuser:vgroup权限；
``` bash
$ chown -R vuser:vgroup /var/www/extsuite/extmail/cgi/
```
### 配置管理后台-ExtMan
yum安装ExtMan
``` bash
$ yum install extsuite-webman
```
更新cgi目录权限 由于SuEXEC的需要，必须将extman的cgi目录修改成vuser:vgroup权限：
``` bash
$ chown -R vuser:vgroup /var/www/extsuite/extman/cgi/
```
链接到基本Extmail
``` bash
$ mkdir /tmp/extman
$ chown -R vuser:vgroup /tmp/extman
```
#### 数据库初始化
启动Mysql
``` bash
$ service mysqld start
$ chkconfig mysqld on
```
导入mysql数据库结构及初始化数据，root密码默认为空
``` bash
$ mysql -u root -p < /var/www/extsuite/extman/docs/extmail.sql
$ mysql -u root -p < /var/www/extsuite/extman/docs/init.sql
```
设置虚拟域和虚拟用户的配置文件
``` bash
$ cd /var/www/extsuite/extman/docs
$ cp mysql_virtual_alias_maps.cf /etc/postfix/
$ cp mysql_virtual_domains_maps.cf /etc/postfix/
$ cp mysql_virtual_mailbox_maps.cf /etc/postfix/
$ cp mysql_virtual_sender_maps.cf /etc/postfix/
```
配置main.cf
``` bash
$ vi /etc/postfix/main.cf
```
增加以下内容
``` bash
# extmail config here
virtual_alias_maps = mysql:/etc/postfix/mysql_virtual_alias_maps.cf
virtual_mailbox_domains = mysql:/etc/postfix/mysql_virtual_domains_maps.cf
virtual_mailbox_maps = mysql:/etc/postfix/mysql_virtual_mailbox_maps.cf
virtual_transport = maildrop:
```
重启postfix：
``` bash
 $ service postfix restart
```
#### 测试authlib
建立刚才导入mysql的postmaster@extmail.org帐户的Maildir，请输入如下命令：
``` bash
$ cd /var/www/extsuite/extman/tools 
$ ./maildirmake.pl /home/domains/extmail.org/postmaster/Maildir 
$ chown -R vuser:vgroup /home/domains/extmail.org
```
执行：
``` bash
$ /usr/sbin/authtest -s login postmaster@extmail.org extmail
```
结果如下：
``` bash
Authentication succeeded.

     Authenticated: postmaster@extmail.org  (uid 1000, gid 1000)
    Home Directory: /home/domains/extmail.org/postmaster
           Maildir: /home/domains/extmail.org/postmaster/Maildir/
             Quota: 104857600S
Encrypted Password: $1$phz1mRrj$3ok6BjeaoJYWDBsEPZb5C0
Cleartext Password: extmail
           Options: (none)
```
这样表明ExtMan的正确安装，数据库也正确导入，courier-authlib能正确连接到mysql数据库
最后访问http://mail.extmail.org/extmail/ ，如无意外，将看到webmail的登陆页，不过此时还没有加正式的用户，所以不能登陆，包括postmaster@extmail.org也不行。必须要登陆到http://mail.extmail.org/extman/ 里增加一个新帐户才能登陆。
ExtMan的默认超级管理员帐户：root@extmail.org，初始密码：extmail*123*，登陆成功后，建议将密码修改，以确保安全。
配置图形化日志
启动mailgraph_ext
``` bash
$ /usr/local/mailgraph_ext/mailgraph-init start
```
启动cmdserver(在后台显示系统信息)
``` bash
$ /var/www/extsuite/extman/daemon/cmdserver --daemon
```
加入开机自启动：
``` bash
$ echo "/usr/local/mailgraph_ext/mailgraph-init start" >> /etc/rc.d/rc.local
$ echo "/var/www/extsuite/extman/daemon/cmdserver -v -d" >> /etc/rc.d/rc.local
```
使用方法： 等待大约15分钟左右，如果邮件系统有一定的流量，即可登陆到extman里，点“图形日志”即可看到图形化的日志。具体每天，周，月，年的则点击相应的图片进入即可。
添加定时任务：
``` bash
$ crontab -e
```
增加以下内容：
``` bash
0 4 * * * /var/www/extsuite/extman/tools/expireusers.pl -all postmaster@extmail.org
30 4 * * * /var/www/extsuite/extman/tools/reportusage.pl -all /home/domains postmaster@extmail.org
### 配置Cyrus-SASL
#### 安装cyrus-sasl
删除系统的cyrus-sasl:
``` bash
$ rpm -e cyrus-sasl --nodeps
```
安装新的支持authdaemon的软件包
``` bash
$ yum install cyrus-sasl
```
配置main.cf文件
Postfix的SMTP认证需要透过Cyrus-SASL，连接到authdaemon获取认证信息。
编辑main.cf
``` bahs
$ vi /etc/postfix/main.cf
```
增加如下内容
``` bash
# smtpd related config
smtpd_recipient_restrictions =
        permit_mynetworks,
        permit_sasl_authenticated,
        reject_non_fqdn_hostname,
        reject_non_fqdn_sender,
        reject_non_fqdn_recipient,
        reject_unauth_destination,
        reject_unauth_pipelining,
        reject_invalid_hostname,

# SMTP sender login matching config
smtpd_sender_restrictions =
        permit_mynetworks,
        reject_sender_login_mismatch,
        reject_authenticated_sender_login_mismatch,
        reject_unauthenticated_sender_login_mismatch

smtpd_sender_login_maps =
        mysql:/etc/postfix/mysql_virtual_sender_maps.cf,
        mysql:/etc/postfix/mysql_virtual_alias_maps.cf
  
# SMTP AUTH config here
broken_sasl_auth_clients = yes
smtpd_sasl_auth_enable = yes
smtpd_sasl_local_domain = $myhostname
smtpd_sasl_security_options = noanonymous
```
编辑smtpd.conf文件
``` bash
$ vi /usr/lib/sasl2/smtpd.conf
```
删除原来内容，并且替换为
``` bash
pwcheck_method: authdaemond
log_level: 3
mech_list: PLAIN LOGIN
authdaemond_path:/var/spool/authdaemon/socket
```
重启postfix:
``` bash
$ service postfix start
```
#### 测试SMTP认证
通过以下命令获得postmaster@extmail.org的用户名及密码的BASE64编码：
``` bash
$ perl -e 'use MIME::Base64; print encode_base64("postmaster\@extmail.org")'
```
内容如下：
``` bash
cG9zdG1hc3RlckBleHRtYWlsLm9yZw==
```
``` bash
$ perl -e 'use MIME::Base64; print encode_base64("extmail")'
```
内容如下：
``` bash
ZXh0bWFpbA==
```
然后本机测试：
``` bash
$ telnet localhost 25
```
其过程如下：
``` bash
Trying 127.0.0.1...
Connected to localhost.localdomain (127.0.0.1).
Escape character is '^]'.
220 mail.extmail.org ESMTP Postfix - by extmail.org
ehlo demo.domain.tld     << 输入内容
250-mail.extmail.org
250-PIPELINING
250-SIZE 10240000
250-VRFY
250-ETRN
250-AUTH LOGIN PLAIN
250-AUTH=LOGIN PLAIN
250-ENHANCEDSTATUSCODES
250-8BITMIME
250 DSN
auth login     << 输入内容
334 VXNlcm5hbWU6
cG9zdG1hc3RlckBleHRtYWlsLm9yZw==     << 输入内容
334 UGFzc3dvcmQ6
ZXh0bWFpbA==     << 输入内容
235 2.0.0 Authentication successful
quit     << 输入内容
221 2.0.0 Bye
```
最后出现235 Authentication Successful 表明认证成功了。
### 配置Courier-IMAP
#### 安装Courier-IMAP
默认的courier-authlib及courier-imap都会增加系统自启动设置，因此下一次服务器启动将自动启动相应的authlib及POP3服务
``` bash
$ yum install courier-imap
```
配置courier-imap
``` bash
$ vi /usr/lib/courier-imap/etc/imapd
```
修改内容如下：
``` bash
IMAPDSTART=NO
```
``` bash
$ # vi /usr/lib/courier-imap/etc/imapd-ssl
```
修改内容如下：
``` bash
IMAPDSSLSTART=NO 
```
然后重启courier-imap
``` bash
$ service courier-imap start
```
测试POP3 请按如下步骤输入pop3命令测试其是否正常工作，注意蓝色的信息是我们输入到POP3服务器的(请首先登录extman自行建立test@extmail.org用户，密码:extmail)
``` bash
$ telnet localhost 110
```
其过程如下：
``` bash
Trying 127.0.0.1...
Connected to localhost.localdomain (127.0.0.1).
Escape character is '^]'.
+OK Hello there.
user test@extmail.org     << 输入内容
+OK Password required.
pass extmail     << 输入内容
+OK logged in.
list     << 输入内容
+OK POP3 clients that break here, they violate STD53.
.
quit     << 输入内容
+OK Bye-bye.
Connection closed by foreign host.
```
## 结束
在搭建完成后，终端下采用ifconfig指令查看本机IP后即可在其他机器上登陆邮件服务系统。








































