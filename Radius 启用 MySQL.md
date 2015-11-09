### Radius 启用 MySQL
反注释 743 行: $INCLUDE sql.conf  

	vi /etc/freeradius/radiusd.conf


编辑 Radius 数据库配置文件 设置数据库类型 用户密码(要和 sql 密码统一).

	vi /etc/freeradius/sql.conf

- 反注释 108行 readclients = yes   来使得 freeradius 从数据库 读取客户端信息.


- 自定义设置(同步修改别的配置文件.)
	- mysql 数据库的端口  37行.
	- radius 数据库的 用户名 密码等内容  39行  
	- 默认:端口 3306  用户名 radius 密码 xujian 



配置 令其使用 mysql 作为数据存储设备

	vim /etc/freeradius/sites-available/default

- authorize{}字段 (69行) 下:
	- 注释 170行 files 行  
		> 这里的file指的就是usrs文件  
		> 将不再把用户信息写在users而使用mysql来存储用户信息
	- 反注释 177行 sql 行  



- accounting{} 字段 (379行) 下:
	- 反注释 407行 的 sql   来启用sql来记录统计信息、

- session{}字段 (451行) 下:
	- 反注释 456行 sql行   启用用户同时登录限制功能.(可选功能)

- post-auth{} 字段 (464行) 下:
	- 反注释 478行 sql 行   启用用户登录后进行数据记录功能.


如果你启动了 启用用户同时登录限制功能 那么接下来还要做这一步

	vim /etc/freeradius/sql/mysql/dialup.conf

找到下面几行(289-293行) 并反注释.
	
	Uncomment simul_count_query to enable simultaneous use checking
	simul_count_query = "SELECT COUNT(*) \
                         FROM ${acct_table1} \
                         WHERE username = '%{SQL-User-Name}' \
                         AND acctstoptime IS NULL"




3、进行测试；
mysql -u root -p
mysql> use radius;
mysql> insert into radcheck(username,attribute,value,op)values("lansgg","Cleartext-Password","password123","=");



重启 freeradius 

radtest lansgg password123 localhost 10 testing123










到这里 mysql 的配置就算完成了.




设置信息汇总:

–MySQL–
localhost: 3306
–Radius–
authentication *: 1812
accounting *: 1813
inner-tunnel authentication 127.0.0.1: 18120
Proxy *:1814
