安装后执行：
`net start mysql` => 启动mysql
`net stop mysql` => 关闭mysql
`mysql -u root -p`  => 指定用户(root)和用户密码，连接数据库
mysql -h 主机名或IP地址 -P 端口号 -u 用户名 -p

退出命令：
简写：\q
标准：ctrl+Z
取消当前未完成的SQL命令：\c

![[Pasted image 20260710153928.png]]

有的版本里不允许修改数据，可以加一行：
SET SQL_SAFE_UPDATES = 0;

操作命令
关键字 
use -> 加仓库名，操作当前仓库下的表格
show -> 展示表格  show tables
insert -> 插入  insert into 表名 （属性名）values(对应属性值)
select -> 查找
	where -> 添加条件
	select * from `[`tableName] where `[keyname] = [keyvalue]
	* 为查询所有符合条件的数据，也可以加上keyname来指定数据展示哪些内容
	like -> 模糊查询（查询值需要用%包起来）
	order by -> 排序  默认正序，后面加DESC为倒序
	<>  -> 表示不等于
delete -> 指定删除表格中的一或几列  delete from `[tableName] where [条件]
update -> 修改表格指定列的信息  update `[tableName]  set [keyname] = [keyvalue]