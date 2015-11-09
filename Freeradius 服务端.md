## 安装 Freeradius 服务端:  

| 安装方式 |                                 |     安装版本    |
|:-------:|:------------------------------- |:--------------|
|自动安装: | sudo apt-get install freeradius | 非最新版        |
|手动安装: |编译源代码                         | 最新版,推荐.    |


### 手动安装教程:
- 去官网查看 最新的版本 
- 下载 源编码包.  [下载链接](ftp://ftp.freeradius.org/pub/freeradius/freeradius-server-3.0.10.tar.gz)
> Debian 下载方法: wget + 下载链接
- 解压压缩包:   tar -xzf 压缩包名
- 进入解压目录 进行编译步骤: 
	- 安装编译radius必需软件 


		可能要先安装 gcc / make 等编译工具 ,按照错误提示 谷歌解决.
		apt-get install libssl1.0.0 libssl-dev
		./configure
		这步可能会出错 错就安装:  apt-get install libtalloc-dev -y
		
		./configure  
		make  
		make install


 卸载软件:如果之前安装过 freeradius 2.0 的话
 apt-get remove freeradius
 apt-get autoremove 自动卸载不需要的软件.
 
 
 

安装完后可运行 radiusd –X ， 进行debug模式启动，若看到最后出现 
Listening on authentication *:1812 
Listening on accounting *:1813 
Ready to process requests. 
则表示可正常运行。
 
 
 Failed binding to auth address 127.0.0.1 port 18120 bound to server inner-tunnel: Address already in use
/usr/local/etc/raddb/sites-enabled/inner-tunnel[33]: Error binding to port for 127.0.0.1 port 18120
 
 
 
  指定FreeRADIUS Server地址，并设置通信密码

cat >>/usr/local/etc/radiusclient/servers<<EOF
localhost   testing123
EOF
注意：这里的通信密码不建议更改！经本人测试，更改后使用不正常。
