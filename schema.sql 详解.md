### schema.sql 脚本内容详解:

> vi /etc/freeradius/sql/mysql/schema.sql

- schema: 架构的意思  也就是给 radius 数据库 设置一系列表(和表的结构)
- 使用方法: 下面命令会 自动加7个表格到 radius 数据库.  
	> mysql -uroot -prootpass radius < schema.sql 


|        表名      |       作用    |
|:---------------|:--------------:|
| radcheck        | 用户检查信息表
| radreply        | 用户回复信息表
| radgroupcheck   | 用户组检查信息表
| radgroupreply   | 用户组检查信息表
| radusergroup    | 用户和组关系表
| radacct         | 计费情况表
| radpostauth     | 认证后处理信息，可以包括认证请求成功和拒绝的记录。