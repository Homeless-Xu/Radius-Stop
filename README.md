## 简介 + 详细安装配置教程


### [FreeRADIUS 官网链接](http://freeradius.org)
###  Radius   [极品教程链接](http://freeradius.akagi201.org/chapter2/installing-FreeRADIUS.html)




<span id="radius"></span>
#### Remote Authentication Dial In User Service    远程用户拨号认证系统

| 命令中心: | 简介 |
|:--------:|:----:|
| /etc/init.d/freeradius start/stop/restart|脚本启动服务|
| freeradius -X  或者  radiusd -X | Debug 模式来启动服务 |
|   



| 配置文件: | 简介 | 本文 |
|:-----------:|:---:|:---:|
| vi /etc/freeradius/radiusd.conf   | 开启 sql 功能  | FreeRADIUS 主配置文件:包含各种子配置文件 
| vi /etc/freeradius/clients.conf   | 添加NAS客户端用      |
| vi /etc/freeradius/sql.conf        | 配置数据库端口 用户名 密码  <br> 开启:使radius 可以从数据库 读取客户端信息. |  3306 radius xujian
| /etc/freeradius                  | freeradius 配置文件目录
| /sbin   | freeradius 运行文件目录


	
|三A框架 | 安全架构模型 |Triple A Framework|
|:-----:|:---------:|:-----------------:|
| authentication | 验证 | 用户是否有权限访问网络. <br> 用户名,密码是否正确 |
| authorization  | 授权 | 哪些用户能使用哪些服务 分配 IP 地址     |
| accounting     | 记账 | 记录用户网络资源的使用情况  |



| | | |
|:------------:|:-------------------:|:-------------------------:|
| 结构: C/S     | Client / Server 端  |   合理分配任务，降低系统开销。 |
| 通信方式: UDP  |  快速但不安全         | 1812端口监听 authentication <br> 1813端口监听 accounting |
	


|   简拼  |        比喻:         | 详解 |
|:------:|:--------------------:|:---:|
| NAS    |        路由器         | 	Network Access Server <br> 提供上网功能的设备 <br> 通过控制实现网络的 合理+安全 使用  <br>可比喻成 VPN服务器 / 路由器 / 智能交换机 |
| Radius | 电信提供拨号上网的服务器 |

##  验证 授权 记账 流程:

|   对象    |                                    |                                                             |
|:--------:|:-----------------------------------:|:----------------------------------------------------------:|
| 用户      | 想上网                               |  向路由器发送接入请求 (输入用户名 密码)                          |
| 路由器    | 收到用户请求                          |  向radius服务器转发用户的接入请求                               |
| Radius   | 收到 路由器 转发的请求                 |  查询数据库,得出结果 . <br> 向 路由器 返回消息包                  |
| 路由器    | 根据 Radius 的返回消息包               |  判断用户能否接入 能就开始上网 不能就断开用户连接.                 |
| 开始上网: | 路由器 → RADIUS: 发送记费开始请求包     |  表明对该用户已经开始记费，                                     |
| 结束上网: | 路由器 → RADIUS: 发送记费停止请求包     |  上网时长,上传下载流量,等使用的网络资源统计                        |





<span id="freeradius"></span>
# FreeRadius：
##### 部署最广泛最实用的开源RADIUS软件. 缺点: 配置复杂.   
##### 服务端 和 客户端 可以是一台设备  也可以是分开的两台设备.
##### 实例  高校:用学号密码 就能登录校园wifi.  只需: Radius 的服务器. 支持 Radius 的无线路由器(能刷 dd-wrt).

|       |   比如  |           功能             |
|:-----:|:------:|:--------------------------:|
| 服务端 |  VPS   | 运行freeradius 服务端软件    |
| 客户端 |  路由器 | 负责将用户信息 传递给服务器    |
| 用户端 |  手机   | 是不需要安装客户端的          |



