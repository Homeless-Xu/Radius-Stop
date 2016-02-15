## Debian 安装教程:



一键安装脚本.

apt-get update ; apt-get install pptpd -y

sed -i 'N;$d' /etc/pptpd.conf  

echo -e "localip 192.168.217.1\nremoteip 192.168.217.234-238,192.168.217.245" \>\> /etc/pptpd.conf


echo xujian pptpd 0219 \* \>\> /etc/ppp/chap-secrets

sed -i -e 's/^#.ms-dns 192.168.1.1/ms-dns 8.8.8.8/g' -i -e 's/^#.ms-dns 192.168.1.2/ms-dns 8.8.4.4/g' /etc/ppp/options

sed -i 's/^#net.ipv4.ip\_forward=1/net.ipv4.ip\_forward=1/g' /etc/sysctl.conf

sysctl -p ; /etc/init.d/pptpd restart ;apt-get install iptables



iptables -t nat -A POSTROUTING -s 192.168.217.0/24 -o eth0 -j MASQUERADE
  
iptables-save \> /etc/iptables.pptp
 
cd /etc/network/if-up.d/ ; touch iptables
 
echo -e "\#\!/bin/sh" -e "\niptables-restore \< /etc/iptables.pptp" \>\> iptables 
 
chmod +x /etc/network/if-up.d/iptables










- 检查并更新本地软件库: 只更新列表 不更新软件?  

		apt-get update   #更新软件库
		apt-get upgrade  #升级软件 少用. -y 直接同意安装.


### 安装 PPTP  

	    apt-get install pptpd
  
### 配置 PPTP
删除文件最后两行.
 
	    sed -i 'N;$d' /etc/pptpd.conf   

添加两行到文件末尾
 
	    echo -e "localip 192.168.217.1\nremoteip 192.168.217.234-238,192.168.217.245" >> /etc/pptpd.conf
  
 
  
> 第一行: vpn 服务器的 IP 地址  随便写，但不要和本地地址冲突就行。​  
> 第二行: 连接 VPN 后,客户端获取到的 IP 地址,(vpn 分配给你的 IP).



### 添加 VPN 帐户  

	echo xujian pptpd 0219 \* >> /etc/ppp/chap-secrets
  
格式: 一行一人 : 用户名 pptpd 用户密码 \*（别忘记最后这个星号） 
   

### 修改 DNS 服务器  

	sed -i -e 's/^#.ms-dns 192.168.1.1/ms-dns 8.8.8.8/g' -i -e 's/^#.ms-dns 192.168.1.2/ms-dns 8.8.4.4/g' /etc/ppp/options
	
	i 参数  直接编辑文件.
	e 参数 同时执行多项编辑.
  
### 开启 IPv4 转发

	sed -i 's/^#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/g' /etc/sysctl.conf

 
### 应用配置 + 重启 PPTP 服务  + 安装 iptables:

	sysctl -p ; /etc/init.d/pptpd restart ;apt-get install iptables


### 开启iptables转发 (分两个命令运行)
> 大概就是 在 iptables 下面的 nat 表 里面添加规则,允许转发.
  
	iptables -t nat -A POSTROUTING -s 192.168.217.0/24 -o eth0 -j MASQUERADE
	
	iptables-save > /etc/iptables.pptp

   

### 开机自动加载 iptables
> 去指定目录; 创建iptables文件 (其实是脚本);  
> 写入内容: #!/bin/sh    iptables-restore \< /etc/iptables.pptp  
> 并给新建脚本运行权限 

	cd /etc/network/if-up.d/ ; touch iptables
	
	echo -e "#\!/bin/sh" -e "\niptables-restore \< /etc/iptables.pptp" >> iptables ;  
	
	chmod +x /etc/network/if-up.d/iptables
  





问题汇总:


连上 vpn  能上 qq 不能上网:








# Ubuntu

pptp vpn 安装:

1. 安装 pptp :        apt-get install pptpd

2. 编辑 pptp.conf     vi /etc/pptpd.conf
3. 将以下内容行的注释去掉：(18 / 99 / 100 行)
option /etc/ppp/pptpd-options
localip 192.168.0.1
remoteip 192.168.0.201-245
这两句设置了当外部计算机通过pptp联接到vpn后所能拿到的ip地址范围和服务器的ip地址设置。



4. 添加登录用户
vi /etc/ppp/chap-secrets
添加一行，格式如下
用户名 pptpd "你想要的密码" \* 用户名不要引号
密码要用半角双引号括起来
千万不能忘了 星号.星号是说允许从任何IP地址联接，如果你想单独设定IP地址也可以。

理论上到这里一个vpn就已经搭建完毕了。无论你用的是Windows还是OSX，或者是iPhone OS，都可以通过建立一个pptp链接来联入这个VPN。




不过你并不能通过这个来上Internet，因为所有的数据都作用于那台pptpd的服务器上， 而不会传入拨入的计算机设备上。要上Internet还需要这么干：

设置DNS解析，编辑pptpd-options
vi /etc/ppp/pptpd-options



























