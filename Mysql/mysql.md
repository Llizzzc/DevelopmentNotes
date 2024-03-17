# MySQL
>1. [千峰教育2021](https://www.bilibili.com/video/BV1qb4y1Y722?p=29&spm_id_from=pageDriver "mysql")
>2. [千峰教育2021高级篇](https://www.bilibili.com/video/BV1h64y1y77i/?spm_id_from=333.788.top_right_bar_window_custom_collection.content.click&vd_source=e5de1dabc977707311263a4bc0f609cc "mysql")

## 一 基本概念
+ 关系型数据库，采用关系模型来组织数据存储，以行和列形式存储数据并记录数据与数据之间的关系（将数据存储在表格），可以通过建立表格与表格之间的关联来维护数据与数据之间的关系。
+ 非关系型数据库，采用键值对模型来存储数据，只完成数据的记录，不会记录数据与数据之前的关系。

## 二 安装配置
+ 在[MariaDB](https://mariadb.org/download/?t=repo-config&d=CentOS+7&v=10.11&r_m=aliyun)查看需要安装版本的仓库，然后配置仓库
+ `yum install MariaDB-server MariaDB-client`	// 安装
+ `systemctl enable mariadb.service`	// 设置自启动
+ `systemctl start mariadb.service`	// 启动
+ `systemctl stop mariadb.service`	// 关闭
+ `mysql -u Username -p`	// 登陆
+ `quit;`	// 退出
+ `show variables like "%character%"; show variables like "%collation%";`	// 查看字符集
+ `cd /etc/my.cnf.d; vim server.cnf`	// 修改字符集
```
[mysqld]
init_connect=‘SET collation_connection = utf8_unicode_ci’
init_connect=‘SET NAMES utf8’
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake
```
+ `cd /etc/my.cnf.d; vim mysql-clients.cnf`	// 修改字符集
```
[client]
default-character-set=utf8
```

## 三 SQL语句分类
### 3.1 DDL：数据定义语言
####  3.1.1 数据库操作
+ `create database (if not exists) dbName (character set Encoding);`	// 创建数据库，可以指定字符编码
+ `show databases;`	// 显示所有数据库
+ `show create database dbName;`	// 显示数据库创建语句
+ `alter database dbName character set Encoding;`	 // 修改数据库字符编码
+ `drop database (if exists) dbName;`	// 删除数据库
+ `use dbName;`	// 切换数据库
#### 3.1.2 表操作
+ 
    ```sql
    create table tbName(
     Term Type (Limits),
     ...
     ...
     primary key(Term, ...)	// 该方式可以定义联合主键
     constraint fkName foreign key(Term) references tbName(Term)	// 定义外键, fkName为逻辑名称, 删除外键约束时使用
    );	// 创建表
    ```
+ `show tables;`	// 显示当前数据库中的表
+ `show create table tbName;`	// 显示数据表创建语句
+ `desc tbName;`	// 显示某表结构
+ `drop table (if exists) tbName;`	// 删除表
+ `alter table tbName rename to NewName;`	// 修改表名
+ `alter table tbName character set Encoding;`	// 修改表字符编码，默认和数据库一致
+ `alter table tbName add Term Type (Limits);`	// 添加字段
+ `alter table tbName change Term NewName Type (Limits);`	// 修改字段名和类型，Limits中的非空约束会重置为允许null，最好一起更改
+ `alter table tbName modify Term Type (Limits);`	// 只修改字段类型
+ `alter table tbName drop Term;`	// 删除字段
+ `alter table tbName drop primary key;`	// 删除表主键约束
+ `alter table tbName drop key Term;`	// 删除表唯一约束
+ `alter table tbName drop foreign key fkName;`	// 删除外键约束
+ `alter table add constraint fkName foreign key(Term) references tbName(Term) on update cascade on delete cascade;`	// 设置外键级联修改和删除，更新被引用表的被引用字段，引用表的引用字段也会同步更新
#### 3.1.3 数据类型
+ 数值类型
  |类型|内存大小|说明|
  |:-:|:-:|:-:|
  |tinyint|1byte||
  |smallint|2byte||
  |mediumint|3byte||
  |int/integer|4byte|常用|
  |bigint|8byte|常用|
  |float|4byte||
  |double|8byte|常用|
  |decimal||（总位数, 小数位数）|
+ 字符串类型
  |类型|字符长度|说明|
  |:-:|:-:|:-:|
  |char|0~255|定长，长度不够会在最后补'\u0000'，常用|
  |varchar|0~65535|不定长，常用|
  |tinyblob|0~255|二进制字符串|
  |blob|0~65535||
  |mediumblob|||
  |longblob|||
  |tinytext|0~255|文本数据|
  |text|0~65535||
  |mediumtext|||
  |longtext||常用|
+ 日期类型
  |类型|格式|说明|
  |:--:|:--:|:--:|
  |date|2021-09-13|常用|
  |time|12:12:33||
  |year|2021||
  |datetime|2021-09-13 12:12:33|常用|
  |timestamp|20210913 121233||
#### 3.1.4 字段约束
+ 非空约束（not null）：必须提供值。
+ 唯一约束（unique）：此字段值不能重复。
+ 主键约束（primary key）：非空且唯一，用来唯一标识一条数据，每个表中最多只能有一个主键，联合主键是将表的多个字段一起设置为表的主键。
+ 自动增长（auto_increment）：只有int类型可以设置，从1开始，只能对一个字段进行设置，且该字段必须为键。
+ 外健约束（foreign key）：建立不同表之间的关系。
### 3.2 DML：数据操作语言
#### 3.2.1 插入语句
+ `insert into tbName(Term, ...) values(Value, ...);`	// 要保持前后对应，有非空约束的字段必须提供值
#### 3.2.2 删除语句
+ `delete from tbName where Condition;`	// 没设置条件则删除表中所有数据
#### 3.2.3 修改语句
+ `update tbName set Term=Value, ... where Condition;`	// 没设置条件则会修改一整列
### 3.3 DQL：数据查询语言
#### 3.3.1 单表查询
+ `select Term, ... from tbName where Condition;`	// 用and、or来支持多条件，用not来取反，条件有=、!=、>、<、>=、<=、between ... and ...、like 'Reg'(%表示任意多个字符，\_为占位符)
+ `select Term from tbName order by Term asc|desc, ...; `	// 结果按字段排序，默认升序，可以先满足第一个规则再加其他规则
+ `select Term as NewName from tbName;`	// 给字段取别名
+ `select distinct Term from tbName;`	// 消除重复
+ `select count(Term) from tbName;`	// 统计字段数
+ `select max|min(Term) from tbName;`	// 指定字段最大值或最小值
+ `select sum(Term) from tbName;`	// 查询指定字段数据总和
+ `select avg(Term) from tbName;`	// 查询指定字段数据平均值
+ `select now()|sysdate();`	// 获取当前系统时间，向datetime字段添加数据可以使用字符串，也可以使用now()和sysdate()
+ `select upper|lower(Term) from tbName;`	// 将指定字段的值转换大小写
+ `select substring(Term, Start, Len) from tbName;`	// 截取字段的值，Start从1开始
+ `select concat(Term1, '-', Term2) from tbName;`	// 拼接字段
+ `select Term, Function(Term) from tbName group by Term having Condition;`	// 按字段分组，返回每个分组的第一个值，可以加上聚合函数一起使用，可加having对分组结果进行筛选
+ `select Term from tbName limit (PageNum - 1) * PageSize, PageSize;`	// 查询第n页数据
#### 3.3.2 多表查询
+ 一对一：通过主键关联，即两张数据表中主键相同的数据为相互对应的数据。也可以通过唯一外键关联，即在任意一张表中添加一个字段设置为外键约束与另一张表的主键关联，同时设置唯一约束。
+ 一对多与多对一：在多端添加外键与一端的主键关联。
+ 多对多：创建一个关系表，在表中设置两个外键分别关联两张表的主键。
+ `select ... from tbName1 inner join tbName2 on Condition;`	// 内连接，生成两张表的笛卡尔积，有很多无用信息，因此要加上匹配条件，可以在表名后加空格取别名简化操作
+ `select ... from tbName1 left join tbName2 on Condition;`	// 左连接，显示tbName1中的所有数据，如果在tbName2中存在与前表满足匹配的数据，则一起显示，否则显示为null
+ `select ... from tbName1 right join tbName2 on Condition;`	// 与左连接相反
+ `select ... from tbName where Term=(select ...);`	// 嵌套查询，将子查询的结果作为父查询的条件，如果子查询返回多个结果，将=换成in，如果是跟在from后面，需要为子查询取别名
### 3.4 DCL：数据控制语言
+ `create user 'UserName'@'Ip' identified by 'Password';`	// 新增用户，Ip指定用户可以在哪个地址登陆，localhost为本地，%为任何地址
+ `rename user 'OldName'@'Ip' to 'NewName'@'Ip';`	// 修改用户名
+ `alter user 'UserName'@'Ip' identified by 'Password';`	// 用户改密
+ `drop user 'UserName'@'Ip';`	// 删除用户
+ `show grants;`	// 查看当前用户权限
+ `show grants for 'UserName'@'Ip';`	// 查看某用户权限
+ `grant Privileges on dbName.tbName to 'UserName'@'Ip' (identified by 'Password') (with grant option);`	// 赋予权限，添加密码表示创建
+ `revoke Privileges on dbName.tbName from 'UserName'@'Ip';`	// 删除权限
+ `flush privileges;`	// 刷新权限
|权限|权限说明|权限级别|
|:--:|:--:|:--:|
|create|创建数据库、表或索引|数据库、表|
|drop|删除数据库或表|数据库、表|
|grant option|赋予权限选项|数据库、表|
|references|创建外键|数据表|
|alter|更改表|数据表|
|delete|删除表数据|数据表|
|index|操作索引|数据表|
|insert|添加表数据|数据表|
|select|查询表数|数据表|
|update|更新表数据|数据表|
|create view|创建视图|视图|
|show view|查看视图|视图|
|alter routine|更改存储过程|存储过程|
|create routine|创建存储过程|存储过程|
|excute|执行存储过程|存储过程|
|file|服务器主机文件的访问|文件管理|
|create temporary tables|创建临时表|服务器管理|
|lock tables|锁表|服务器管理|
|create user|创建用户|服务器管理|
|reload|执行flush privileges, refresh, reload等命令|服务器管理|
|process|查看进程|服务器管理|
|replication client|查看主从服务器状态|服务器管理|
|replication slave|主从复制|服务器管理|
|show database|查看数据库|服务器管理|
|shutdown|关闭数据库|服务器管理|
|super|超级权限	|服务器管理|
|all|所有||
|usage|只能登陆||

## 四 存储过程
### 4.1 概念
将能够将完成特定功能的SQL指令进行封装，编译之后存储在数据库服务器上，并为其取一个名字，客户端就可以通过名字来直接调用这个SQL指令集，获取执行结果。
### 4.2 优缺点
#### 4.2.1 优点
+ SQL指令无需客户端编写，通过网络传送，可以节省网络开销，同时避免SQL指令在网络传输过程中被恶意篡改。
+ 存储过程经过编译创建并保存在数据库中，执行过程无需重复的进行编译操作，对SQL指令的执行过程进行了性能提升。
+ 存储过程中多个SQL指令之间存在逻辑关系，支持流程控制语句，可以实现更为复杂的业务。
#### 4.2.2 缺点
+ 存储过程是根据不同的数据库进行编译、创建并存储在数据库中，当我们需要切换到其它数据库产品时，需要重新编写存储过程。
+ 存储过程受限于数据库产品，如果需要性能优化会成为一个问题。
+ 在互联网产品中，如果需要数据库的高并发访问，使用存储过程会增加数据库的连接执行时间，因为我们把复杂的业务交给了数据库处理。
### 4.3 使用
#### 4.3.1 存储过程管理
存储过程是属于某个数据库的，只能在当前数据库调用存储过程。
+ ```sql
	create procedure prcName(in InArg Type, ..., out OutArg Type, ...)
	begin
		...
	end;	// 创建存储过程
	```
+ `delimiter Symbol;`	// 定义sql结束符号，默认是分号
+ `call prcName(InArg, ..., OutArg);`	// 调用
+ `show procedure status where db='dbName';`	// 查看某数据库的存储过程
+ `show create procedure dbName.prcName;`	// 查看存储过程创建细节
+ `alter procedure prcName Property;`	// 修改存储过程特征
	+ contains sql	// 包含SQL语句，但不包含读写语句
	+ no sql	// 不包含SQL语句
	+ reads sql data	// 包含读语句
	+ modifies sql data	// 包含写语句
	+ sql security definer|invoker	// 指明定义者还是调用者可执行
	+ comment 'Descript'	// 注释
+ `drop procedure prcName;`	// 删除存储过程
#### 4.3.2 变量
+ `declare Arg Type (default Value);`	// 定义局部变量，只能在存储过程中定义
+ `set @Arg=Value;`	// 定义用户变量，变量名以@开头，作用域为当前会话，尽量少使用，set也可以进行赋值
+ `select @Arg;`	// 查看用户变量
+ `select count(Term) into Arg from tbName;`	// 将查询结果赋值给变量
#### 4.3.3 参数
+ `in`	// 输入参数，可以有多个
+ `out`	// 输出参数，可以有多个
+ `inout`	// 可以当输入也可以当输出
#### 4.3.4 流程控制
+ ```sql
	if Condition then
		...
	else
		...
	end if;
	```
+ ```sql
	case Arg
	when Condition then
		...
	when Condition then
		...
	else
		...
	end case;
	```
+ ```sql
	while Condition do
		...
	end while;
	```
+ ```sql
	repeat
		...
	until Condition
	end repeat;
	```
+ ```sql
	LoopName:loop
		...
		if Condition then
			leave LoopName;
		end if;
	end loop;
	```
### 4.4 游标
在存储过程中，需要返回多条数据时，游标可以用来依次取出查询结果集中的每一条数据，要结合循环语句使用。
+ `declare cursName cursor for select ...;`	// 声明游标
+ `open cursName;`	// 打开游标
+ `fetch cursName into Arg ...;`	// 提取游标当前指向的数据，游标自动下移
+ `close cursName;`	// 关闭游标

## 五 触发器
### 5.1 概念
特殊的存储过程，但是触发器无需调用，执行DML操作时会自动触发执行，无需手动调用，可以使用NEW和OLD关键字在触发器中获取触发这个触发器的DML操作的数据。
### 5.2 优缺点
#### 5.2.1 优点
+ 触发器是自动执行的。
+ 可以实现表中数据的关联操作，有利于保证数据的完整性。
+ 可以对DML操作的数据进行更为复杂的合法性校验。
#### 5.2.2 缺点
+ 使用触发器实现的业务逻辑如果出现错误将难以定位，后期维护困难。
+ 大量使用触发器易造成代码结构杂乱，增加程序的复杂性。
+ 当触发器操作的数据量比较大时，执行效率会大大降低。
+ 互联网项目中应避免使用触发器，对于并发量不大的项目可以使用存储过程，但是不提倡。
### 5.3 使用
+ ```sql
	create trigger triName	// 创建触发器
	before|after	// 触发时机
	insert|delete|update	// 触发操作
	on tbName
	for each row	// 操作几条记录就触发几次
	...;
	```
+ `show triggers;`	// 查看触发器
+ `drop trigger triName;`	// 删除触发器
+ new.Term：在触发器中用于获取insert操作添加的数据、update操作修改后的数据
+ old.Term：在触发器中用于获取delete操作删除前的数据、update操作修改前的数据

## 六 视图
### 6.1 概念
由数据中一张表或多张表根据特定的条件查询出的数据构成的虚拟表。
### 6.2 作用
+ 安全性：如果我们直接将数据表授权给用户操作，那么用户可以CRUD任何数据，假如我们想要对数据表中的部分数据进行保护，可以将公开的数据生成为视图，授权用户访问视图。
+ 简单性：如果我们需要查询的数据来源于多张表，可以使用多表连查，我们通过视图将这些连查的结果对用户开放，用户则可以通过视图获取多表数据。
### 6.3 使用
+ ```sql
	create view viName	// 创建视图
	as
	select ... from ...;
	```
+ `select ... from viName;`	// 从视图查询数据
+ `desc viName;`	// 查询视图结构
+ `create or replace view viName as select ...;`	// 修改视图
+ `alter view viName as select ...;`	// 修改视图
+ `drop view viName;`	// 删除视图，不会影响原数据表中的数据
### 6.4 特性
+ 查询操作：如果在数据表中添加了新的数据，而且这个数据满足创建视图时查询语句的条件，通过查询视图也可以查询出新增的数据，当删除原表中满足查询条件的数据时，也会从视图中删除。
+ 增加操作：如果在视图中添加数据，数据会被添加到原数据表。
+ 删除操作：如果在视图中删除数据，数据也会从原表删除。
+ 修改操作：如果在视图中修改数据，也会修改原数据表的数据。

## 七 索引
### 7.1 概念
将数据表中的某一列或者某几列的值取出来构造成便于查找的结构进行存储，生成数据表的目录。当我们进行数据查询的时候，则先在目录中进行查找得到对应数据的地址，然后再到数据表中根据地址快速获得数据记录，避免全表扫描，Mysql中索引使用B+Tree实现。
### 7.2 优缺点
#### 7.2.1 优点
+ 索引大大降低了数据库服务器在执行查询操作时扫描的数据。
+ 索引可以避免服务器排序，将随机IO变成顺序IO。
#### 7.2.2 缺点
+ 索引是根据数据表字段创建的，当数据表中数据发生DML操作时，索引需要更新。
+ 索引文件也会占用磁盘空间。
### 7.3 分类
+ 主键索引：在数据表的主键字段创建的索引，声明为主键时自动创建。
+ 唯一索引：在数据表的唯一字段创建的索引，声明唯一约束时自动创建，可自行创建。
+ 普通索引：普通字段上创建的索引。
+ 组合索引：两个及以上字段联合起来创建的索引，建议不超过过五个字段，使用最左前缀法则判断是否命中。
+ 全文索引：进行查询的时候，数据源可能来自不同的字段或者不同的表，MyISAM存储引擎支持全文索引，实际开发中，一般使用第三方的搜索引擎中间件如ElasticSearch。
### 7.4 使用
+ `show indexes|keys from tbName;`	// 查询数据表索引
+ `create (unique) index indexName on tbName(Term, ...);`	// 创建索引
+ `drop index indexName on tbName;`	// 删除表索引
+ `explain select ...\G`	// 查看查询语句细节，在命令行执行
+ `show engines`	// 查看支持的存储引擎
### 7.5 存储引擎
+ MyISAM(非聚集索引)：把索引和数据放在两个文件中，查到索引后还要去另一个文件中找数据，性能会慢一些，除此之外，天然支持表锁和全文索引。
+ InnoDB(聚集索引)：把索引和数据存放在一个文件中，通过找到索引后就能直接在索引树上的叶子节点中获得完整的数据，可以实现行锁和表锁。

## 八 事务
### 8.1 概念
完成同一个业务的多个DML操作。
### 8.2 特性
+ 原子性(Atomicity)：一个事务中的多个DML操作，要么同时执行成功，要么同时执行失败。
+ 一致性(Consistency)：事务执行前后，数据库中的数据是一致的，完整性和一致性不能被破坏。
+ 隔离性(Isolation)：多个并行的事务之间不能相互影响。
+ 持久性(Durability)：事务完成之后，对数据的操作是永久的。
### 8.3 事务管理
+ 在MySQL中，默认的DML指令是自动提交的，当我们执行一个DML指令后，自动同步到数据库中。
+ 手动提交需要执行`start transaction;`开启事务，如果事务执行过程中出现异常，使用`rollback;`来撤销操作，否则使用`commit;`来进行提交。
### 8.4 事务隔离级别
+ 读未提交(read uncommitted)：事务2可以读取事务1执行但未提交的数据，可能会导致脏读(读取未提交数据)。
+ 读已提交(read committed)：事务2只能读取事务1已提交的数据，避免了脏读，可能会导致虚读(在同一个事务中，两次查询操作读取到的数据不一样)。
+ 可重复读(repeatable read)：事务2第一次查询之后，在其结束之前其它事务不能修改对应的数据，避免了虚读，但可能导致幻读(事务2对数据表的数据进行修改然后查询，在查询之前事务1向数据表新增了一条数据，就导致事务2以为修改了所有数据，但却查询出了与修改不一致的数据)。
+ 串型化(serializable)：同时只允许一个事务对数据表进行操作，避免了脏读、虚读、幻读问题。
### 8.5 设置隔离级别
+ `select @@tx_isolation|transaction_isolation;`	// 查看隔离级别，默认为可重复读
+ `set session transaction isolation level Rank;`	// 设置隔离级别

## 九 数据库设计流程
1. 根据应用系统的功能，分析数据实体
	+ 电商系统：商品、用户、订单...
	+ 管理系统：学生、课程、成绩...
2. 提取实体的数据项
	+ 商品：名称、图片、描述...
	+ 用户：姓名、密码...
3. 检查数据项是否满足三范式，不满足三范式可能会导致数据的冗余，造成维护困难
	+ 一范式：要求数据表的字段不可再分
		![](./img/04.png)
	+ 二范式：不存在非关键字段对关键字段的部分以依赖
	  ![](./img/01.png)
	+ 三范式：不存在非关键字段之间的传递依赖
		![](./img/02.png)
		![](./img/03.png)
4. 绘制E-R图
5. 数据库建模
	+ 三线图
	+ PowerDesigner
	+ PDMan
6. 建库建表
7. 数据测试

## 十 SQL优化
### 10.1 优化目的和途径
SQL优化的目的是为了SQL语句能够具备优秀的查询性能，实现这样的目的有很多的途径。
+ 工程优化实现：数据库标准、表结构标准、字段标准、创建索引。
+ SQL语句的优化：当前SQL语句有没有命中索引。
### 10.2 Explain工具
得知当前系统里有哪些SQL是慢SQL（超过1s），然后通过explain工具可以对当前SQL的性能进行判断。要知道哪些是慢SQL，有两种方式：
+ 开启本地Mysql的慢查询日志。
+ 云部署的Mysql服务器，提供了查询慢SQL的功能。
#### 10.2.1 各个列细节说明
##### 10.2.1.1 select_type
`set session optimizer_switch='derived_merge=off';`	// 关闭衍生表的合并优化，用来查看衍生表
该列描述了查询的类型：
+ simple：简单查询
+ primary：外部的主查询
+ derived：在from后面进行的子查询，会产生衍生表
+ subquery：在from前面进行的子查询
+ union：联合查询
##### 10.2.1.2 table
该列表示SQL正在访问哪一张表，包括衍生表。
##### 10.2.1.3 type
该列可以直观的判断出当前的SQL语句的性能，type里的取值和性能的优劣顺序如下：
null > system > const > eq_ref > ref > range > index > all，对于SQL优化来说，要尽量保证type列的值是属于range及以上级别。
+ null：性能最好，一般在使用了聚合函数操作索引列，结果直接从索引树获取即可
+ system：很少见，直接和一条记录进行比较
+ const：使用主键索引或者唯一索引和常量进行匹配
+ eq_ref：在进行多表连接查询时，查询条件是使用了主键进行比较
+ ref：查询条件是普通索引
+ range：使用索引进行范围查找
+ index：查询没有进行条件判断，但是所有数据都可以直接从索引树上获取
+ all：全表扫描
##### 10.2.1.4 id
在多个select中，id越大的越先执行，如果id相同，上面的先执行。
##### 10.2.1.5 possible keys
这一次查询中可能会用到的索引，也就是说Mysql内部优化器会进行判断，如果这一次查询走索引比全表扫描的性能要差，那么这次查询就走全表扫描，这样的判断依据可以通过trace工具来查看。
##### 10.2.1.6 key
这一次查询中实际使用的索引。
##### 10.2.1.7 rows
该SQL语句可能要查询的数据条数。
##### 10.2.1.8 key_len
可以看出当前命中了联合索引的哪几列，长度是根据字段类型计算的。
##### 10.2.1.9 Extra
该列展示了这条SQL的一些其他信息，能够帮助我们判断当前SQL是否使用了覆盖索引、文件排序等信息。
+ Using index：使用了覆盖索引，所谓的覆盖索引，指的是当前查询的所有字段都是索引列，这就意味着可以直接从索引列中获取数据，而不需要查表，这种优化手段是经常用的
+ Using where：使用了普通索引做查询条件
+ Using index condition：没有使用覆盖索引，建议可以使用覆盖索引来优化
+ Using temporary：在非索引列上进行去重操作就需要使用一张临时表来实现
+ Using filesort：使用文件排序，会使用磁盘+内存的方式进行文件排序
+ Select tables optimized away：直接在索引列上进行聚合函数的操作，没有进行任何表的操作
#### 10.2.2 Trace工具
+ `set session optimizer_trace='enabled=on', end_markers_in_json=on;`	// 开启trace，第二个变量只在Mysql中存在
+ `select * from information_schema.OPTIMIZER_TRACE;`	//	获取trace分析结果
### 10.3 Mysql内部优化器
在SQL查询开始之前，Mysql内部优化器会进行一次自我优化，让这一次的查询性能更好。
+ `show warnings;`	// 查看内部优化之后的结果
### 10.4 文件排序原理
在执行文件排序的时候，会把查询的数据的大小与系统变量max_length_for_sort_data（默认1024字节）进行比较，如果比系统变量小，那么执行单路排序，反之执行双路排序：
+ 单路排序：把所有数据放进sort_buffer内存缓冲区中进行排序
+ 双路排序：取数据的排序字段和主键字段，在内存缓冲区中排序完成后，将主键字段做一次回表查询，获取完整数据

## 十一 锁
### 11.1 定义
锁是用来解决多个任务（线程、进程）在并发访问同一共享资源时带来的数据安全问题，虽然使用锁解决了数据安全问题，但是会带来性能的影响，频繁使用锁的程序的性能是必然很差的。
### 11.2 分类
#### 11.2.1 从性能上划分
+ 悲观锁：悲观的认为当前的并发是非常严重的，所以在任何时候操作都是互斥的，保证了线程安全，但牺牲了并发性。
+ 乐观锁：乐观的认为当前并发并不严重，因此对于读的情况，大家都可以进行，但是对于写的情况，再进行上锁。以CAS自旋锁为例，在某种情况下性能是可以的，但是频繁自旋会消耗很大的资源。
#### 11.2.2 从数据的操作细粒度上划分
+ 表锁：对整张表上锁。
+ 行锁：对表中的某一行上锁。
#### 11.2.3 从数据库的操作类型上划分
+ 读锁（共享锁）：对同一行的数据进行读来说，是可以同时进行，但是写不行。
+ 写锁（排他锁）：在上了写锁之后，以及释放写锁之前，在整个过程中是不能进行任何的其他并发操作。
### 11.3 表锁
对整张表进行上锁。MyISAM存储引擎是天然支持表锁的，也就是说在MyISAM的存储引擎中如果出现并发的情况，将会出现表锁的效果。MyISAM不支持事务。
+ `lock table tbName read|write;`	// 对表上锁
+ `show open tables;`	// 查看当前会话的表上锁情况
+ `unlock tables;`	// 释放当前会话的所有锁
### 11.4 行锁
MyISAM只支持表锁，但不支持行锁，InnoDB可以支持行锁。有以下方式：
+ 在并发的事务里，每个事务的增删改操作相当于是上了行锁
+ `select ... for update;`	// 第二种上行锁方式
### 11.5 MVCC设计思想
Mysql在读和写的操作中，对读的性能进行了并发性的保障，让所有读都是快照读，对于写的时候，进行版本控制，如果真实数据版本比快照版本新，那么写之前就要进行版本更新，这样就可以既提高读的并发性，又保证写的安全性。
### 11.6 死锁
所谓的死锁，就是开启的锁没有办法关闭，导致资源的访问因为无法获得锁而处于阻塞状态。
### 11.7 间隙锁
行锁只能对某一行上锁，间隙锁表示对一个范围进行上锁。