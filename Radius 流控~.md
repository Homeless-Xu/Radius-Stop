# 文章目录
### [ 安装 FreeRADIUS 工具箱(含测试工具)](#工具包) --- apt-get install freeradius-utils
	
### [安装 Mariadb 数据库(mysql 替代版)]() 

### [ 配置 客户端 ](#客户端)
### [ 取消注释 steve段(81-89行) :开启测试 ](#开启测试)
	
		vi /etc/freeradius/users 

### 改共享密码 :  Cleartext-Password := "xujian"

- 查看防火墙状态:   iptables -L 
- 如有需要 修改防火墙配置 允许freeradius所使用的端口1812，1813通过。
	
		iptables -A INPUT -p udp --dport 1812 -j ACCEPT
		iptables -A INPUT -p udp --dport 1813 -j ACCEPT
		

###  新建数据库.以及导入 一系列表.
mysql -u root –p
要求输入密码时，直接回车即可。
	
	create database radius;  
	
	grant all on radius.* to radius@localhost identified by "radpass"; 
	
	exit;

导入表结构

	mysql -u root -p radius < /etc/freeradius/sql/mysql/schema.sql


### 建立 用户和组 
输入mysql -u root -p   输入密码 进数据库.
use radius，使用该数据库
1. 建立组信息：（在此新建组名称为user）
insert into radgroupreply (groupname,attribute,op,value) values ('user','Auth-Type',':=','Local'); 
insert into radgroupreply (groupname,attribute,op,value) values ('user','Service-Type',':=','Framed-User'); 
insert into radgroupreply (groupname,attribute,op,value) values ('user','Framed-IP-Address',':=','255.255.255.255'); 
insert into radgroupreply (groupname,attribute,op,value) values ('user','Framed-IP-Netmask',':=','255.255.255.0');

2. 建立用户信息：（在此新建用户名为test，密码为testpwd）
insert into radcheck (username,attribute,op,value) values ('test','Cleartext-Password',':=','testpwd');

3. 将用户加入组中： 可以在 radusergroup 表里面看到.
insert into radusergroup (username,groupname) values ('test','user');


- [配置 sql.conf](#sql.conf)
- [安装 服务端](#服务端安装)
- [配置 VPN服务器](#配置VPN服务器)
- [配置 PPTPd](#配置pptpd)
- - [详解 ](#)
- 启用 Mysql
- 





# 配置客户端: 
 安装服务端 就自带客户端的.!!!!!!  版本 1.1.7 
 

vi /etc/freeradius/clients.conf，  
添加如下内容，注意将IP_ADDRESS和SHARED_SECRET换为你的服务器IP和你的共享密钥。

client IP_ADDRESS {
    ipaddr = IP_ADDRESS
    secret = SHARED_SECRET
    require_message_authenticator = no
}






配置RadiusClient
RadiusClient用于将pptpd和xl2tpd的radius插件的验证请求发送给FreeRadius服务器，充当nas的角色，其配置文件位于/etc/radiusclient内。




客户端可以有很多个

每个 客户端 可设置单独的 共享密码.


 
 





------------------------------------------------------------------------------

# 安装配置流程简介:


1. 安装主要软件及依赖包.  mysql php 
2. 新建mysql数据库
3. 编辑 sql.conf        修改 sql radius数据库 登录密码.
- 编辑 radius.conf    让 freeradius 可以用 sql 功能.
- 编辑  clients.conf  设置新的共享密码



- apt-get update   更新软件列表.
- apt-get upgrade  更新软件.
- apt-cache search radiusclient  搜索软件.  

安装 FreeRADIUS  软件 和 其他组件.

| 软件包 |  简介 | 要求 |
|:---:|:---:|:---:|
| freeradius |  服务器端程序  |   必须装
| freeradius-utils | 客户端程序 必装 包括: <br> 要用到 radtest 测试是否正常 | radclient <br> radtest , radzap  <br> radsniff , smbencrypt |
| freeradius-mysql  | mysql 插件 | 必选 |
| phpmyadmin          | 选择: apache2 <br> 选择: 配置 no |  必选 |
| mysql-server | 
| freeradius-dialupadin | web管理插件 |
| libfreeradius2 | FreeRADIUS的共享库 | 
| php5-gd  php5-curl | php-pear php5-mail-mime php-db | 






 
 
 


	 	
<span id="通信密码"></span>	 	 	
### 修改默认通信密码	 
- radius: udp 协议传输的. 安全性能不太好  所以就有了通信密码

- 通信密码:用于加密 freeradius 服务器 和 客户端 通信的信息.  共享 key	
- ** 强烈建议修改更改通信密码  ( 第101行 →  默认的 testing123 改为 xujian ) **  
		
		vi /etc/freeradius/clients.conf 			
		改掉后如果测试有问题 重启服务器电脑使修改内容生效. 
			
	 			 	



##### 	启用测试账号,测试 freeradius 是否正常工作 
 	
	 	vi /etc/freeradius/users   ( 注释掉 76行 )
	 	
	 	# steve Cleartext-Password := "testing"
 		
 		
 - FreeRADIUS自带一个默认的测试客户用户叫做localhost. 用来帮助排查问题和测试.   
	
	 	
- 用 启动脚本来关闭FreeRADIUS来确保没有其他实例在运行. 
	
		/etc/init.d/freeradius stop

- 以debug模式启动FreeRADIUS. 
	
		freeradius -X

	屏幕上的最后一行是:  Ready to process requests.  
	说明安装正确, 如果这个没有发生, 阅读FreeRADIUS的debug输出.		
	 		 	

- 本地电脑 另建终端 ssh登服务器 执行

		radtest steve testing localhost 1812 xujian 
	
|命令|解析|
|:---:|:---:|  
| radtest |  测试命令
| steve testing | 测试用户名 测试密码
| localhost  1812 | freeradius 运行的 IP 地址和 端口号
| xujian | 通信密码  
	
			
	
	 测试成功 有如下行

		rad_recv: Access-Accept packet from host 127.0.0.1 ......	
	 	

	
	-------




<span id="配置VPN服务器"></span>

# 配置VPN服务器
为pptpd和xl2tpd启用radius插件，首先查找插件的位置。

sudo updatedb
locate radius.so
插件通常位于/usr/lib/pppd/2.4.5/radius.so。


<span id="配置pptpd"></span>
### 配置pptpd
查看/etc/pptpd.conf，获取pptpd的ppp配置文件位置

option /etc/ppp/pptpd-options
修改/etc/ppp/pptpd-options文件，在最后添加如下两行，注意修改插件的具体位置。

	plugin /usr/lib/pppd/2.4.5/radius.so
	radius-config-file /etc/radiusclient/radiusclient.conf
重启pptpd

sudo service pptpd restart







<span id="添加用户"></span>	
## 添加用户:

|
|:---
| 登陆 MySQL 数据库  --------------------------------------------------------------------------------------------- mysql -uroot -p;
| 使用 radius 数据库 --------------------------------------------------------------------------------------------------- USE radius;
| 向 radchec表添加用户 (用户名 xujian，密码 xujian ) :
| INSERT INTO radcheck (username,attribute,op,VALUE) VALUES ('xujian','Cleartext-Password',':=','xujian');
|将用户 xujian 加入VIP1用户组:
| INSERT INTO radusergroup (username,groupname) VALUES ('xujian','VIP1');
 
 
	

默认freeradius只允许本地客户端发起认证， 
这里假设有一台提供VPN服务的服务器名字是red，  
ip为198.74.49.76，设置的共享密码为xujian，  

其中IP必须为客户端IP，这里 vps 服务器 既当服务端 又当 客户端.

shortname是可以随意设置的，客户端连接的时候会被要求提供secret：

vi /etc/freeradius/clients.conf


client IP_ADDRESS {
  ipaddr = 198.74.49.76
  secret = xujian
  require_message_authenticator = no
  }

	
	
	
	
	
	编辑 clients.conf，确定哪个 IP 范围可以访问 Radius 服务，这里 secret 一行的 testing123 就是上面 DD-WRT 要用到的 WPA Shared Key：

$ sudo vi /etc/freeradius/clients.conf

...
client 192.168.2.0/24 {
        require_message_authenticator = no
        secret          = testing123
        shortname       = vpsee-network
}
...



client 192.168.1/24 {
  shortname = netgear
  secret = xujian
  require_message_authenticator = no
  }
  
  
  
  
  
  
  
  
  
  
  
  
  
  
# clients.conf
  
  客户端配置目录: 增加 NAS 客户端到这里才能连接.
  
  
  127.0.0.1  也叫 localhost    默认是开启的 方便你测试 radius 是否能正常工作.
  
  每个 客户端 都有个 shortname  用来区分别的客户端.
  
  1.0 版本   client 后面跟的是 客户端的 ip.
  2.0 版本   ip 由中括号 里面的 ipaddr 来定义了.
  
  例子:
  
  client localhost {
        #  ip 地址. 或者 ip 组!!!!!
           ipaddr = 127.0.0.1/24
默认 子网掩码 是 32    最好不要改.


# 共享密码:

必须改.

老的客户端  在发起连接请求时候 可能不发送 Message-Authenticator 
require_message_authenticator = no   关闭这个功能 就好了            待确定.


short name   也就是 ip 的 别名  为了区别不同客户端用
客户端1.版本 需要 
     2 版本 不需要.
     
     
     
     
     现在可以给 每个客户端 都设定各自的密码.
比如:
     
client 192.168.0.0/24 {
       secret          = testing123-1
       shortname       = private-network-1
}

client 192.168.0.0/16 {
       secret          = testing123-2
       shortname       = private-network-2
}
     


只允许固定的 ip 能连接服务器.
clients per_socket_clients {
       client 192.168.3.4 {
               secret = testing123
        }
}       



         
  
   配置 默认表:
   
  vi /etc/freeradius/sites-available/default
  
  
  确保 pam 行 没被注释掉
  
  authenticate {
        Auth-Type PAP {
                pap
        }
        Auth-Type CHAP {
                chap
        }
        Auth-Type MS-CHAP {
                mschap
        }
        pam
        unix
        eap
}
...
  
  
  
  
  
  同样的确定 inner-tunnel 配置文件里面的 pam 一行没有被注释掉：
  
  vi /etc/freeradius/sites-available/inner-tunnel
  
  authenticate {
        Auth-Type PAP {
                pap
        }
        Auth-Type CHAP {
                chap
        }
        Auth-Type MS-CHAP {
                mschap
        }
        pam
        unix
        eap
}





在 users 加入下面认证类型：

$ sudo vi /etc/freeradius/users
...
DEFAULT Auth-Type = PAM
      Fall-Through = 1
...




# 报错汇总:
  
  freeradius -X 
  
  错误:  Failed binding to authentication address * port 1812: Address already in use
  
  
  原因: freeradius 已经在运行.  没必要再启动. 
  
  
  
  查看 linux 系统运行 进程. 命令   ps -A 显示所有进程.
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  配置流程:    小改 改一次 测试一次
  用 freeradius -X  来测试      通过再然后进行小改.
  
  
  
  
  
  

	
ddwrt 路由器 当 radius 客户端:
将WLAN无线路由器的 IP 地址添加到 RADIUS 服务器上。
在WLAN无线路由器中启用并指定 RADIUS 服务器。
将 RADIUS 用户名或组名添加到您的策略中。
要在配置中启用并指定 RADIUS 服务器。

	
	
	
	
	
	
	
	
	




	
	
	
	
	
	
	
#  MySQL 集成	









#### admin.sql 脚本内容详解:
	
	CREATE USER 'radius'@'localhost';  
	SET PASSWORD FOR 'radius'@'localhost' = PASSWORD('xujian');
	GRANT SELECT ON radius.* TO 'radius'@'localhost';
	GRANT SELECT ON radius.* TO 'radius'@'localhost';
	GRANT ALL on radius.radacct TO 'radius'@'localhost';
	GRANT ALL on radius.radpostauth TO 'radius'@'localhost';

--------
	
	CREATE USER 'radius'@'localhost';  
>- 给数据库 新建用户
>- radius: 新用户名
>- @localhost:  只能通过服务器本机进行登录. 不能通过网络远程登录!!!!!!!
	
	SET PASSWORD FOR 'radius'@'localhost' = PASSWORD('xujian');
> - 给数据库新用户设置密码
> - 密码 要在单引号内

	GRANT SELECT ON radius.* TO 'radius'@'localhost';
> - 给新建用户  数据库(radius)的 查询 权限.

	GRANT ALL on radius.radacct TO 'radius'@'localhost';
> - 给新建用户  数据库(radius)的 radacct 表的所有权限.

	GRANT ALL on radius.radpostauth TO 'radius'@'localhost';
> - 给新建用户  数据库(radius)的 radpostauth 表的 所有权限.







## nas.sql 脚本内容详解:
- 在一个NAS可以使用FreeRADIUS服务器之前,   
必须先在FreeRADIUS服务器上的定义这个客户端.


给数据库添加一个表 网络设备表.   

使用方法: 
> mysql -uroot -p radius < nas.sql
	
	
	CREATE TABLE nas (
  	id int(10) NOT NULL auto_increment,
 	nasname varchar(128) NOT NULL,
  	shortname varchar(32),
  	type varchar(30) DEFAULT 'other',
  	ports int(5),
  	secret varchar(60) DEFAULT 'secret' NOT NULL,
  	server varchar(64),
  	community varchar(50),
  	description varchar(200) DEFAULT 'RADIUS Client',
  	PRIMARY KEY (id),
  	KEY nasname (nasname)
	);



解析:
















freeredius -X
如果调试出错. 把调试文件 想办法 保存成文件.
然后仔细看
我的最后几行错误是:
/etc/freeradius/sql.conf[22]: Instantiation failed for module "sql"
/etc/freeradius/sites-enabled/default[177]: Failed to find "sql" in the "modules" section.
/etc/freeradius/sites-enabled/default[69]: Errors parsing authorize section.



其实重点是 : 你只看了 结果  具体错误 是显示在中间的.电脑太快 你没看的很多内容其实很重要

 
MYSQL check_error: 1054 received 
rlm_sql (sql): database query error, SELECT id, nasname, shortname, type, secret, server FROM nas: Unknown column 'server' in 'field list'


跟sql模块没关系，是你的数据库结构被破坏了

Server字段不存在

手动在数据库的nas里添加一个server字段就搞定了







 grant select,insert,update,delete on mydb.* to [email=test2@localhost]test2@localhost[/email] identified by “abc”;
 用户 test2  密码 abc
 只能在 localhost( 本机 也就是装了 mysql 数据库的那台电脑 也就是别人无法通过网络登录 mysql 只能在服务器电脑上进行登录 ) 上登录 只能对 mydb 数据库 进行查询 插入 修改 删除

设空密码的话 :
grant select,insert,update,delete on mydb.* to [email=test2@localhost]test2@localhost[/email] identified by “”;








作用:
为 RADIUS 创建  默认的 管理员账号和密码
给服务器



应该 该里面的  localhost 和 radpass   不要用默认的.





 
- GRANT ON: 增加新用户并控制其权限  命令格式：
- grant 操作(查询/插入/修改/删除) on 数据库名称.* to 用户名@登录主机 identified by “密码”;
- 例子: 
	
		grant select,insert,update,delete on *.* to [email=test1@”%]test1@”%[/email]” Identified by “abc”;
		
		select,insert,update,delete  查询 插入 修改 删除 权限
		*.* 所有数据库 . 所有数据表?
		







1. 创建 radius 数据库
	
		create database radius default character set utf8;
		
- 安装 freeradius 专用 mysql 插件

		sudo apt-get install freeradius-mysql

	- 安装后, 就多了如下文件夹( 包含几个 sql 脚本 ):  


			cd /etc/freeradius/sql/mysql 
	
	   
	   - admin.sql 脚本:  创建 Radius 数据库的默认管理员
	   		- 默认创建的 账户的名称和密码是 radius / radpass
	   
	   - schema.sql 脚本:  自动创建 radius 数据库里的表.
	   
	   修改 radius 数据库的管理员密码 (两个地方同步修改)
	 
	 - 在admin.sql里修改管理员密码，
	  	
			vi /etc/freeradius/sql/mysql/admin.sql
	  		 
	  	    15行 PASSWORD('radpass') 改成 PASSWORD('xujian');
	  		 
	 - 同步修改 sql.conf 的管理员密码.    
	  	
	  		vi /etc/freeradius/sql.conf
	  		
	  		39行 改成 password = "xujian"
	  		
	  		





- 导入 admin.sql 和 schema.sql  

> - 下面的命令 要 cd /etc/freeradius/sql/mysql  才能执行的.  
> - 其实就是 mysql里创建用户密码 和 表 .
> - 如果你重复配置好多次 可能会报错.比如已经建了默认的账户密码.
> - 要改密码的话 这个 只能手动去mysql 手动改密码啦.
> - 表重复的话 应该没关系的.
> - 账号密码 都在 mysql里(软件自带的表) 

 		mysql -uroot -p < admin.sql  
 		
 		mysql -uroot -p radius < schema.sql
 
 
 
 
 
 


Authentication通常是考虑的第一步, 
这是一个过程用来确认是否Alice提供的credential是有效的. 提供credential的最常见的方式是通过用户名和密码. 其他方式像one-time token, certificate, PIN numbers, 或者甚至biometric scanning也可以使用.

在成功authentication之后一个session被初始化. 这个session持续直到网络连接被终止掉.






共享密钥(shared secret)

客户端和服务器端也需要有一个共享的密钥, 将会用来加密和解密某些AVPs. User-Password AVP的值使用这个共享密钥加密. 当共享密钥在客户端和服务器端不同时, FreeRADIUS服务器将会检测到他并且当运行的debug模式时警告你.






---------  



- 然后在/usr/sbin目录下创建连接：
	
		sudo ln -s freeradius radiusd


- FreeRadius的配置文件位于/etc/freeradius目录。

- 编辑 radius 用户配置文件 添加服务器 ip 和共享密码 xujian

		vi /etc/freeradius/clients.conf
		
		添加下面内容到文件
		
		client IP_ADDRESS {
    	ipaddr = 198.74.49.76
   	 	secret = xujian
   		require_message_authenticator = no
		}






整合 pptp 与 freeradius
首先 搭建 pptp 的服务器. 
要整合 还需要一个  radius 的客户端. 如果路由器有 radius 功能 就不需要整合了.






客户端:
radius 认证库:   apt-get install freeradius-utils 



配置 vpn 服务器 

1. pptpd 启用 radius 插件.

首先查找插件位置(后面用到)  
sudo updatedb      更新索引数据库?
locate radius.so   查找 插件位置. 得到

	/usr/lib/pppd/2.4.6/radius.so


查看 /etc/pptpd.conf 文件 找到 18行  得到 ppp的配置的文件位置. 

编辑查找出来的 ppp-option 文件

		vi /etc/ppp/pptpd-options

文件最后添加 (插件路径 用你第一步找到的)
plugin /usr/lib/pppd/2.4.6/radius.so
radius-config-file /etc/radiusclient/radiusclient.conf


重启 pptpd
sudo service pptpd restart





##DaloRadius
用 php 编写的 一个 web 管理 freeradius 的程序. 


下载文件 
wget http://sourceforge.net/projects/daloradius/files/daloradius/daloradius-0.9-8/daloradius-0.9-8.tar.gz




安装必要 php 组件

	apt-get install php-pear php-db


然后新建一个文件夹   
mkdir /usr/share/daloRadius

解压下载的文件
tar zxvf daloradius-0.9-8.tar.gz

复制文件到新建目录  mv daloradius-0.9-8/* /usr/share/daloRadius/ 

删除下载文件 
rm -r daloradius-0.9-8




这时我们将daloRADIUS中附带的sql文件导入MySQL数据库，别忘了输入密码：
	
	mysql -uroot -p radius < /usr/share/daloRadius/contrib/db/fr2-mysql-daloradius-and-freeradius.sql



安装 并解压
cd /tmp
下载:
wget http://sourceforge.net/projects/daloradius/files/daloradius/daloradius0.9-9/daloradius-0.9-9.tar.gz
 
 


编辑 daloRADIUS的配置文件   

	vi /usr/share/daloRadius/library/daloradius.conf.php
首先是MySQL登录信息：

$configValues['CONFIG_DB_HOST'] = 'localhost';
$configValues['CONFIG_DB_USER'] = 'radius';
$configValues['CONFIG_DB_PASS'] = '***';  // 设为自己的密码
$configValues['CONFIG_DB_NAME'] = 'radius';



下面有一个daloRADIUS的bug，默认配置中有一个表名和我们导入的不一样，把它改过来：

$configValues['CONFIG_DB_TBL_RADUSERGROUP'] = 'radusergroup';



然后修改daloRADIUS的路径：
$configValues['CONFIG_PATH_DALO_VARIABLE_DATA'] = '/usr/share/daloRadius/var';


] 添加Apache虚拟主机，如果有Web控制面板什么的自然就方便多了，
不然就编辑/etc/httpd/conf/httpd.conf，加入：

Alias /vpn "/usr/share/daloRadius/"
<Directory "/usr/share/daloRADIUS">
</Directory>


 重启重启Apache和MySQL：


service httpd restart
service mysqld restart



 打开浏览器，进入daloRADIUS的管理页面
 （本例中为http://your.domain/vpn），使用默认用户名administrator和密码radius登录。


。在Management中添加一个新用户，注意密码类型选择Cleartext-Password。

 在终端里再次启动radius -X，同时在另一个终端中用radtest username password localhost 0 MyVPN测试一下，看看现在是不是还能正常接通，如果没问题就OK，让我们把这套系统接驳到PPP上。
 
 
 
 





### 配置 pptpd
编辑/etc/ppp/options.pptpd，里面已经有许多配置选项了，我们要保证有下面的几行，如果没有就添加上去，为了保障用户登录的安全我们限制只使用MS-CHAPv2：

refuse-pap
refuse-chap
refuse-mschap
require-mppe-128
require-mschap-v2


在配置文件最后加上3行：
plugin radius.so
plugin radattr.so
radius-config-file /usr/local/etc/radiusclient/radiusclient.conf

启动服务

一切完成后我们不需要使用debug模式启动radiusd了：

service radiusd start

 当然，我们可以把radiusd和pptpd设为开机启动服务：
 chkconfig radiusd on
 chkconfig pptpd on
 
 至此，PPTP+FreeRADIUS+MySQL+daloRADIUS全部配置完毕，我们在本机上使用添加的用户名和密码拨入VPN，可以正常使用。在daloRADIUS中，还可以看到各个用户每次连接的时长，上传和下载的数据量统计等。daloRADIUS其他的使用方法，本文不再叙述。
 
 
 





locate 文件搜索.

updatedb 文件搜索的 索引库.x






配合 mysql.


使用 radius 需要配置 http php mysql 环境,
用 apache 作为http 服务器.



 


## Debian 安装 L2TP

一、安装 IPSec

IPSec 会对 IP 数据包进行加密和验证。  

推荐用 openswan 这个后台软件包来跑 IPSec。

安装 openswan:

sudo apt-get  install openswan



1.1 配置sysctl参数

 

   sudo mv /etc/sysctl.conf /etc/sysctl.conf.bak

备份文件 

    sudo vim /etc/sysctl.conf

    —————新建文件 并输入如下内容-------------

    net.ipv4.ip\_forward = 1

    net.ipv4.conf.all.accept\_redirects = 0

    net.ipv6.conf.all.accept\_redirects = 0

    net.ipv4.conf.all.send\_redirects = 0 









# Mac 电脑 vpn 连接方法

设置 →  网络 → 左下角 加号 → 接口名称 VPN 服务名称:随便取 → 创建 

配置:默认

服务器地址:vps 服务器的 ip 外网 ip . 固定 ip

账户名: 设置的 VPN 账户名  

加密: 自动

鉴定设置:选择密码   输入VPN密码

高级: 通过 vpn 连接发送所有流量  打钩.

点击连接 . ok



## VPN 内外网 同时使用方法

1、自行搭建 VPN 服务器或购买 VPN 提供商的服务。

2、打开系统偏好设置—\>网络，增加 VPN 设置，VPN 类型选择PPTP，

根据提示设置用户名密码等信息即可。

3、下载 chnroutes.py，相关网址：Downloads - chnroutes - Scripts to generate special routes for china ips

4、打开终端进入下载文件的目录，执行：python chnroutes.py -p mac，该目录下会生成两个文件「ip-up」和「ip-down」。

5、把这两个文件复制到 /etc/ppp 下，然后进入该目录执行：sudo chmod a+x ip-up ip-down



测试一下，在终端执行：netstat -nr，检查路由表的输出信息。

连接 VPN，  再次执行：netstat -nr，你会发现路由表已经发生了变化。

这时再去访问国内的网站，你发现他们又变得嘻唰唰了，同时还能访问 Twitter 和 Facebook！







## FreeRADIUS

官网 :  [http://freeradius.org][1]

debian 7 下教程:

[https://ttboa.wordpress.com/2014/09/26/freeradius-on-debian-7/][2]

服务器: Debian 8:

安装freeradius:     sudo apt-get install freeradius

开启测试账号密码:(用完后 记得删掉.) 

vim /etc/freeradius/users

76-84 行 前的# 去掉.

testuser Cleartext-Password := "testpassword"

重启服务   service freeradius restart

查看服务状态:  service freeradius status

进行测试

radtest steve testing localhost 0 testing123



如果看到Access-Accept就说明连接成功了。

连接失败

radtest testuser testpassword localhost 0 testing123

radclient: no response from server for ID 25 socket 3











1. 下载ppp源码，因为要用到其中的配置文件：

\`\`\` 
1   # wget ftp://ftp.samba.org/pub/ppp/ppp-2.4.5.tar.gz
2   # tar zxvf ppp-2.4.5.tar.gz
3   # cp -R /root/ppp-2.4.5/pppd/plugins/radius/etc/ /usr/local/etc/radiusclient
\`\`\`

1. 编辑/usr/local/etc/radiusclient/servers，加上一组服务器和密钥，本例中为“MyVPN”： IT网，http://www.it.net.cn
   \`\`\` 
   1   # vim /usr/local/etc/radiusclient/servers
   2   localhost MyVPN
   \`\`\`
2. 编辑/usr/local/etc/radiusclient/dictionary，将最后一行改为：

INCLUDE /usr/local/etc/radiusclient/dictionary.microsoft IT网，http://www.it.net.cn

可以再添加一行：

INCLUDE /usr/local/etc/radiusclient/dictionary.merit IT网，http://www.it.net.cn

1   # vim /usr/local/etc/radiusclient/dictionary

2   INCLUDE /usr/local/etc/radiusclient/dictionary.microsoft

3   INCLUDE /usr/local/etc/radiusclient/dictionary.merit

1. 编辑/etc/raddb/clients.conf，把client localhost段下的secret改成刚才指定的密钥。 Linux学习，http:// linux.it.net.cn
2. 编辑/etc/raddb/radiusd.conf，找到$INCLUDE sql.conf，去掉前面的#；找到$INCLUDE sql/mysql/counter.conf，去掉前面的#。
3. 添加MySQL用户及数据库，你可以使用现成的phpMyAdmin等工具，也可以在终端下操作。本例中，创建了radius的用户和同名的数据库：

\`\`\` 
1   # mysql -uroot -p
2   (如果有密码，请输入)
3   \> CREATE USER 'radius'@'localhost' IDENTIFIED BY '\*\*\*';
4   \> CREATE DATABASE IF NOT EXISTS `radius` ;
5   \> GRANT ALL PRIVILEGES ON `radius` . \* TO 'radius'@'localhost';
\`\`\`

1. 编辑/etc/raddb/sql.conf，配置login（用户名），password（密码），radius\_db（数据库名）等字段，并找到readclients一行，设为yes并去掉注释符号#。
2. 编辑/etc/raddb/sites-enabled/default，根据下面的说明注释或取消注释相应的行：

    authorize段，关掉files，打开sql，也可以把unix关掉

    preacct段，关掉files

    accounting段，打开sql，也可以把unix关掉

    session段，打开sql

    post-auth段，打开sql

    pre-proxy段，关掉files IT网，http://www.it.net.cn

到这一步，我们的FreeRADIUS就算配置好了，用户信息都将保存在MySQL数据库中。至于数据库中的表，我们在后面统一导入。

配置daloRADIUS

1. 首先下载并安装daloRADIUS，其中需要安装一个Pear-DB的包：
   \`\`\` 
   1   # wget http://sourceforge.net/projects/daloradius/files/daloradius/daloradius-0.9-8/daloradius-0.9-8.tar.gz
   2   # pear install DB
   3   # mkdir /usr/share/daloRadius
   4   # tar zxvf daloradius-0.9-8.tar.gz
   5   # mv daloradius-0.9-8/\* /usr/share/daloRadius/
   6   # rm -r daloradius-0.9-8
   \`\`\`
2. 这时我们将daloRADIUS中附带的sql文件导入MySQL数据库，别忘了输入密码： Linux学习，http:// linux.it.net.cn
   \`\`\` 
   1   # mysql -uroot -p radius \< /usr/share/daloRadius/contrib/db/fr2-mysql-daloradius-and-freeradius.sql
   \`\`\`
3. 编辑/usr/share/daloRadius/library/daloradius.conf.php，这是daloRADIUS的配置文件。首先是MySQL登录信息：
   \`\`\` 
   1   # /usr/share/daloRadius/library/daloradius.conf.php
   2   $configValues['CONFIG\_DB\_HOST']() = 'localhost';
   3   $configValues['CONFIG\_DB\_USER']() = 'radius';
   4   $configValues['CONFIG\_DB\_PASS']() = '\*\*\*'; // 设为自己的密码
   5   $configValues['CONFIG\_DB\_NAME']() = 'radius';
   \`\`\`
   下面有一个daloRADIUS的bug，默认配置中有一个表名和我们导入的不一样，把它改过来：
   $configValues['CONFIG\_DB\_TBL\_RADUSERGROUP']() = 'radusergroup';
   然后修改daloRADIUS的路径：
   $configValues['CONFIG\_PATH\_DALO\_VARIABLE\_DATA']() = '/usr/share/daloRadius/var';
4. 添加Apache虚拟主机，如果有Web控制面板什么的自然就方便多了，不然就编辑/etc/httpd/conf/httpd.conf，加入： Linux学习，http:// linux.it.net.cn
   \`\`\` 
   1   # vim /etc/httpd/conf/httpd.conf
   2   Alias /vpn "/usr/share/daloRadius/"
   3   \<Directory "/usr/share/daloRADIUS"\>
   4   \</Directory\>
   \`\`\`
5. 重启重启Apache和MySQL： Linux学习，http:// linux.it.net.cn
   \`\`\` 
   1   # service httpd restart
   2   # service mysqld restart
   \`\`\`
6. 打开浏览器，进入daloRADIUS的管理页面（本例中为http://your.domain/vpn），使用默认用户名administrator和密码radius登录。
   daloRADIUS似乎写的不怎么样，最新稳定版已经是三年之前的了，不过作者直到现在还在更新SVN，下次有机会可以用最新的SVN版本试试看。在Management中添加一个新用户，注意密码类型选择Cleartext-Password。 IT网，http://www.it.net.cn
7. 在终端里再次启动radius -X，同时在另一个终端中用radtest username password localhost 0 MyVPN测试一下，看看现在是不是还能正常接通，如果没问题就OK，让我们把这套系统接驳到PPP上。
   配置pppd
8. 编辑/etc/ppp/options.pptpd，里面已经有许多配置选项了，我们要保证有下面的几行，如果没有就添加上去，为了保障用户登录的安全我们限制只使用MS-CHAPv2： IT网，http://www.it.net.cn
   \`\`\` 
   1   # vim /etc/ppp/options.pptpd
   2   refuse-pap
   3   refuse-chap
   4   refuse-mschap
   5   require-mppe-128
   6   require-mschap-v2
   7    
   8   # 在配置文件最后加上3行：
   9       
   10  plugin radius.so
   11  plugin radattr.so
   12  radius-config-file /usr/local/etc/radiusclient/radiusclient.conf
   \`\`\`
   启动服务
9. 一切完成后我们不需要使用debug模式启动radiusd了：
   \`\`\` 
   1   # service radiusd start
   \`\`\`
10. 当然，我们可以把radiusd和pptpd设为开机启动服务：
	\`\`\` 
	1   # chkconfig radiusd on
	2   # chkconfig pptpd on
	\`\`\`
	至此，PPTP+FreeRADIUS+MySQL+daloRADIUS全部配置完毕，我们在本机上使用添加的用户名和密码拨入VPN，可以正常使用。在daloRADIUS中，还可以看到各个用户每次连接的时长，上传和下载的数据量统计等。daloRADIUS其他的使用方法，本文不再叙述。
	文章来源于互联网，具体链接一时忘记未有记下。。。如下几点则为具体配置中所出现的问题及解决方法

1、执行第十三点时报如下错误：

pear not found

经谷歌，解决方法如下：

1   ＃ yum install php-pear

2   ＃ pear upgrade PEAR

2、daloRadius登录界面，用户名为administrator，没记错的话密码是radius。修改密码到config-\>Opeartors修改。 Linux学习，http:// linux.it.net.cn

3、MySQL错误：Database connection error Error Message: DB Error: extension not found

解决方法：

1   ＃ yum install php-mysql -y

4、无法连接MySQL：Database connection error Error Message: DB Error: connect failed

1   # vim /usr/share/daloRadius/library/daloradius.conf.php

2   # $configValues['CONFIG\_DB\_HOST']() = '127.0.0.1'; # 默认为 localhost

5、无法拨入，客户端报691错误： IT网，http://www.it.net.cn

1   ＃ vim /usr/local/etc/radiusclient/dictionary

2   #INCLUDE /etc/radiusclient/dictionary.microsoft

3   INCLUDE /usr/local/etc/radiusclient/dictionary.merit

4   INCLUDE /usr/local/etc/radiusclient/dictionary.microsoft

5   INCLUDE /usr/local/etc/radiusclient/dictionary.ascend

6   INCLUDE /usr/local/etc/radiusclient/dictionary.compat

 

PS：添加流量控制：

1、建立表格

1   \> INSERT INTO radgroupreply (groupname,attribute,op,VALUE) VALUES ('user','Auth-Type',':=','Local');

2   \> INSERT INTO radgroupreply (groupname,attribute,op,VALUE) VALUES ('user','Service-Type',':=','Framed-User');

3   \> INSERT INTO radgroupreply (groupname,attribute,op,VALUE) VALUES ('user','Framed-IP-Address',':=','255.255.255.255');

4   \> INSERT INTO radgroupreply (groupname,attribute,op,VALUE) VALUES ('user','Framed-IP-Netmask',':=','255.255.255.0');

5   \> INSERT INTO radgroupreply (groupname,attribute,op,VALUE) VALUES ('user','Acct-Interim-Interval',':=','600');

6   \> INSERT INTO radgroupreply (groupname,attribute,op,VALUE) VALUES ('user','Max-Monthly-Traffic',':=','5368709120');

7   \> INSERT INTO radgroupcheck (groupname,attribute,op,VALUE) VALUES ('user','Simultaneous-Use',':=','1');

以上前四行不用改动，acct-interim-interval是计算流量的间隔（600秒），意味着每隔10分钟记录当前流量。倒数第二行是每月最大流量，这里是5G（单位是字节）。最后一行是允许同时连接数目。 IT网，http://www.it.net.cn

2、修改配置

1   # vim /etc/raddb/sql/mysql/dialup.conf

2   sql\_user\_name = "%{%{Stripped-User-Name}:-%{%{User-Name}:-none}}"

3   # 取消前面的注释并注释下一行

4   # 同时如果需要打开simultanoues-use（控制同时在线用户数）的话需要把simul\_query\_check取消注释。

3、输入测试用户信息：

1   \> INSERT INTO radcheck (username,attribute,op,VALUE) VALUES ('test','Cleartext-Password',':=','test');

2   \> INSERT INTO radusergroup (username,groupname) VALUES ('test','user');

(用户名与密码必须以明文/NTLM Crypt形式保存，因为MS-CHAPv2不支持MD5保存的密码。) IT网，http://www.it.net.cn

4、由于上步中有非内置的attribute Max-Monthly-Traffic，所以需要在/usr/local/etc/raddb/dictionary里面定义：

ATTRIBUTE Max-Monthly-Traffic 3003 integer IT网，http://www.it.net.cn

5、初步测试： IT网，http://www.it.net.cn

1   # /etc/init.d/freeradius stop

2   # radiusd -X \>

3   # radtest test test localhost 1649 testing123

如果结果中有Access-Accept就代表成功了，否则退回去检查设置。

6、测试成功的话添加在认证时检测流量的语句，打开/usr/local/etc/raddb/sites-enabled/default，找到authorize一节插入：

1   # vim /usr/local/etc/raddb/sites-enabled/default

2       update request {

3       Group-Name := "%{sql:SELECT groupname FROM radusergroup WHERE username='%{User-Name}' ORDER BY priority}"

4       }

5       if ("%{sql: SELECT SUM(acctinputoctets+acctoutputoctets) FROM radacct WHERE username='%{User-Name}' AND date\_format(acctstarttime, '%Y-%m-%d') \>= date\_format(now(),'%Y-%m-01') AND date\_format(acctstoptime, '%Y-%m-%d') \<= last\_day(now());}" \>= "%{sql: SELECT value FROM radgroupreply WHERE groupname='%{Group-Name}' AND attribute='Max-Monthly-Traffic';}") {

6       reject

7       }

7、设置radiusclient：

打开/usr/local/etc/radiusclient/radiusclient.conf，修改

authserver localhost:1812

acctserver localhost:1813

修改server：

最后加上localhost secret（就是FreeRadius的client.conf里面定义的secret）

修改dictionary：

最后加上 IT网，http://www.it.net.cn

INCLUDE /usr/local/etc/radiusclient/dictionary.merit

INCLUDE /usr/local/etc/radiusclient/dictionary.microsoft Linux学习，http:// linux.it.net.cn

如果没有dicitionary.microsoft的话看这里：http://wiki.freeradius.org/PopTop

8、启动服务

1   # service radiusd start

2   # chkconfig radiusd on

3   # chkconfig pptpd on

至此，完成了所有配置了。。。 Linux学习，http:// linux.it.net.cn

(责任编辑：IT)







客户端 安装客户端  1.1.7 

[http://freeradius.org/freeradius-client/][10]

接下来 是防火墙的联动 

这个暂且不详细说，每个厂商的防火墙配置方法都不同，思路是差不多的，客户端如果想完成认证，必须通过radius认证，这个时候，pc的数据包到达防火墙后，防火墙会把客户端丢过来的认证信息发送到radius服务器上面，由radius服务器去辨别是否通过认证

  6.radius里关键的配置文件为radius.conf 与clients.conf

               radius.conf:这个文件记录的是radius服务相关的信息，包括服务器监听的接口、允许认证的客户端IP等

               clients.conf:定义客户端与服务器之间的共享密钥，认证信息等






















RADIUS包有一个指定的格式,两个关键的组件是:

code: 表示包的类型.
attribute: 携带RADIUS使用的必要数据.


##### 数据包
了解一个RADIUS数据包的格式将会大大帮助理解RADIUS协议. 

让我们更细致的查看RADIUS数据包.
 我们将会看一个简单的authentication请求. 
 一个客户端发送一个Access-Request包到服务器. 服务器返回一个Access-Accept包来表示成功.

显示在这里的RADIUS数据包只是一个UDP数据包的payload. UDP和IP协议的讨论已经超出这本书的范围了.





Radius 数据包: 
第一个字节是 code  表示 radius 包的类型.
第二个字节是 identifier   唯一标识符. 由客户端生成.
大概就是客户端 发送的每个包 都有个固定的 唯一标识符. 以便服务器能通过唯一标识符 给你回复信息.
第三,第四个字节 是 length
表示数据包中的内容 一直到哪里是有用的 这个边界外的字节 被认为是填充的可忽略的.
 
 
 
 
 
 
每个数据包 都是由 code 来识别的  code 占一个字节  特定的code 的值 代表这个包的特性和需求

这个消息提供数据来改变一个先有用户session的authorization. 例如, 我们现在可以动态改变每个session的带宽限制. 这让我们可以当互联网连接下降时可以增加每个session的带宽. 


| RADIUS <br> code (decimal) | Packet type | Sent by |
|:---:|:---------------------------|:---------------:|
| 1   |	Access-Request	            | NAS             |
| 2	  | Access-Accept	            | RADIUS server   |
| 3	  | Access-Reject	            | RADIUS server   |
| 4	  | Accounting-Request          | NAS             |
| 5	  | Accounting-Response	         | RADIUS server   |
| 11  |	Access-Challenge          	 | RADIUS server    |
| 12  |	Status-Server(Experimental)	 |                  |
| 13  |	Status-Client(Experimental)	 |                  |
| 255 |	Reserved	                 |                  |
|     |                             |                  |
| 40  |	Disconnect-Request	         | RADIUS server    |
| 41  |	Disconnect-ACK	             | NAS              |
| 42  |	Disconnect-NAK	             | NAS              |
| 43  |	CoA-Request	                 | RADIUS server    |
| 44  |	CoA-ACK	                     | NAS              |
| 45  |	CoA-NAK	                     | NAS              |








[跳到页首](#header)
