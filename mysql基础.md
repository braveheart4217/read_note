# Mysql 基础

and 优先级比 or 高

order by binary   binary 强制开启大小写敏感
还有order by 默认是升序排列，使用desc降序排列

CURRENT_TIMESTAMP 获取当前时间戳

select 
form（database_name.table_name）
where
(注意子句的的使用顺序，如果顺序不对也有可能出错)
group by
having
order by
limit

别名：select 后面查询的资料取一个自己想要的名字
select {栏位|资料|运算式} [AS] [别名] （别名可加可不加）(note:别名请用单引号或者双引号包上)
	
	eg: select count(*) as 'all' from fileInfo 

算术运算符

	<=> 表示等于,其特殊支出在与判断NULL值得时候(见下面的is)
逻辑运算符的优先顺序 NOT > AND > OR,使用的时候要注意运算符优先级
其他运算符 

BETWEEN ... AND ... 
>**note**:比如 BETWEEN 300 AND 400 应该是 x >=300 and x <= 400
  
IN(...)
>select * from fileInfo where id in (1,2,3)

IS (IS NOT)
>select * from fileInfo where dependence is NULL 
>select * from fileInfo where dependence <=> NULL
>select * from fileInfo where dependence = NULL(错误)

LIKE
> % 表示0到多个任意字符
> _ 表示一个字符
		
### order by
order by后面可以指定多个排序的字段（多个字段可以有自己的排序规则），也可以用字段别名，运算式来作为排序字段
>select * from fileInfo order by name DESC ,md5 ASC ,path DESC
>
>其会先按照name排序，如果有条目一样会继续按照md5排序，以此类推
>
>select id，MAX(path) as 'max_path' from fileInfo order by 'max_path'

limit 返回指定数目的数据
>limit 3    只返回查询数据的前三条
>
>limit 3，3 返回所以查询数据的 4-6 条
>
>note:如果出现类似 limit 10000000，10的语句查询效率会比较低（因为检索了10000010条数据但是只返回了10条数据）


select [ALL | distinct(DISTINCT)] name from fileInfo
>请注意这两条语句的区别

	SELECT DISTINCT count(name) from fileInfo order by name
	SELECT COUNT(*) from (SELECT DISTINCT name from fileInfo order by name ) `tmp`

###数值

整数范围： -![](http://i.imgur.com/Y34QP4I.gif) 到 ![](http://i.imgur.com/Y34QP4I.gif) - 1
小数范围： 包含小数的数字，整数范围与上面一样，小数位最多30位

![](http://i.imgur.com/CR82YLP.png)

要执行字符串的连接工作可以使用 || 运算符，其在条件判断中表示 或，但也可以用作字符串的连接操作，但是默认情况下此功能不开启，如果需要可以设置
	set sql_mode = 'PIPES_AS_CONCAT';

函数
函数名与左括号之间不能有空格

更多函数请参考[Mysql参考手册](https://dev.mysql.com/doc/refman/5.7/en/string-functions.html)

条件判断

	SELECT  name , if(size > 47956,'big','small') as size from fileInfo

case关键字

	case
		when 'de' then 'des'
		when 'deee' then 'desee'
		else 'time'   //default value
	end

或者

	case
		when condition= 'de' then 'des'
		when condition= 'deee' then 'desee'
		else 'time'   //default value
	end

IFNULL(参数，表达式) 如果参数为NULL，就返回表达式的值，否则返回参数的值
ISNULL(参数) 如果参数为NULL，就返回TRUE，否则返回FALSE

COUNT(参数) 函数在参数为NULL的时候计算当前条目到总数

	select count(*),count(name),count(dependece),count(assetNames);

意思就是说如果在结果集中的某一行记录的name/dependence/AssetNames为空那就不会计入总数

GROUP_CONCAT函数
![](http://i.imgur.com/YWET3ar.png)

GROUP BY (可以通过指定多个分组字段来结局不同平台下文件名相同的问题(上次的解决办法是通过增加唯一标识字段(name_platformType)))
![](http://i.imgur.com/QvqnqEl.png)

如果带上 `with rollup` 会在返回的结果中为每一个分组增加一条统计结果

群组函数不能用在`where`子句中，因为群组函数的作用对象是群组，`where`子句是返回结果集的条件，在结果集没有返回之前如果进行群组操作，然后如果要使用群组函数就需要在 `HAVING` 子句中使用

`JOIN and UNION`多表联合查询

多表查询方式一：使用where
指定表格名称：如果在多表查询中表中出现重复名称的字段，这时候就需要为这些重名字段指明表名

note：仅仅那些重名的字段需要指明表名

	select country.Code,country.Capital,city.Name
	from country,city
	where country.Capital = city.ID;

表格别名：仅仅在表名后面跟上别名就好了

![](http://i.imgur.com/pWKVEQI.png)

多表查询方式二：使用`INNER JOIN`(如果作为条件的字段名字相同可以使用USING(fieldName) )
	select Code,Capitail,city.Name
	from country INNER JOIN city 
	ON
		Capitail = ID

在使用 INNER JOIN 的时候要注意一点的就是，一定要符合 **结合条件**的条目才会出现，以上面的下面例子为例

	SELECT empno,ename,e.deptno,d.dname
	from cmdev.emp e,cmdev.dept d
	where e.deptno = d.deptno
如果emp表中的deptno字段有在dept中不存在的或者emp表中有条目的deptno字段为空，那么这些不符合 结合条件 的资料就不会出现在结果集中，此时如果你先让这些条目出现在结果集中就应该使用`OUTER JOIN`

`OUTER JOIN` 有 `LEFT` 与 `RIGTH`两种结合方式两种结合方式的区别就是以哪边表格为主，比如rigth方式就是以右边的表为主，右边表的所有数据都会出现在结果集中，不管是否满足 结合条件，如下：

	SELECT empno,ename,e.deptno,d.dname
	from cmdev.emp e left join cmdev.dept d
 	on e.deptno = d.deptno

	SELECT empno,ename,e.deptno,d.dname
	from cmdev.emp e right join cmdev.dept d
 	on e.deptno = d.deptno

虽然只有`left`与`right`的差别，但是满足的需求确实完全不同的


###合并查询
合并查询指的是将一次或者多次查询的结果合并成一个，使用`UNION`关键字可以满足以上要求

		SELECT * from fileInfo where id = 1
	union 
		SELECT * from fileInfo where id = 2

使用UNION查询的时候有一些规则需要准守与知道

- 回传结果的栏位名称是第一个规则的选取的字段
- 所有查询的字段的数量一定要一样


数据插入：`insert (into)`
![](http://i.imgur.com/7a38Xb2.png)
如下一次插入多条数据(限mysql)

	insert into cmdev.emplog(logno,logdt,message) 
	values
		(3,CURRENT_DATE,'tim3w'),
		(4,CURRENT_DATE,'tim4w'),
		(2,CURRENT_DATE,'tim3');

或者
	
	insert into cmdev.emplog
	set logno = 5,
		logdt =  CURRENT_DATE,
		message = 'nimei';

如果你还可以在使用 `INSERT` 的时候发生主键冲突的时候选择加入 `IGNORE` 关键字，他可以在发生主键冲突的时候自动忽略此条记录

	insert IGNORE into cmdev.emplog(logno,logdt,message) 
	values
		(3,CURRENT_DATE,'tim3w'),

如果你想在插入时发生主键冲突时更新对应的记录可以使用：`ON DUPLICATE KEY UPFATE`

	INSERT into cmdev.travel
	values (7900,'BOSTON',1)
	on duplicate key update counter = counter + 1;

或者你可以使用`REPLACE`来新增记录

	replace into cmdev.travel
	values (7900,'BOSTON',1);
在没有违反主键冲突的时候效果与`insert`是一样的，但是replace当发生主键冲突的时候默认会执行更新操作，即用当前值覆盖过去的值

### UPDATE
 ![](http://i.imgur.com/JGvq9Si.png)

同样在UPDATE的一条记录的主键的时候发生主键冲突同样也可以使用`INGOER`关键字以达到忽略的效果，但是可能使用ignore之后的执行结果可能会跟你想象的不同，例如：

	update ignore fileInfo set size='fuck', name='test' where id = 1;

这条语句的执行效果是将 name 设置成 test，然后将size设置成 0(size为整形)，因为size数据类型是整形，但语句试图出入一个字符串，所以会出错，但是使用了ingore之后其被设置成了0，但是 name 的值是正确的，所以name被设置成test，结果确实与想象的不同


### delete
 ![delete操作](http://i.imgur.com/PYOCbdQ.png)

	delete from fileInfo [where ...]
	truncate table fileInfo
这两者都会删除所以的记录，但是truncate的执行效率较高 


字符集与数据库

关于`character set`与`collate`(字符序)概念的理解

- 字符`(Character)`是指人类语言中最小的表义符号。例如’A'、’B'等；

- 给定一系列字符，对每个字符赋予一个数值，用数值来代表对应的字符，这一数值就是字符的编码(Encoding)。例如，我们给字符’A'赋予数值0，给字符’B'赋予数值1，则0就是字符’A'的编码；

- 给定一系列字符并赋予对应的编码后，所有这些字符和编码对组成的集合就是字符集(Character Set)。例如，给定字符列表为{‘A’,'B’}时，{‘A’=>0, ‘B’=>1}就是一个字符集；

- 字符序(Collation)是指在同一字符集内字符之间的比较规则；

- 确定字符序后，才能在一个字符集上定义什么是等价的字符，以及字符之间的大小关系；

- 每个字符序唯一对应一种字符集，但一个字符集可以对应多种字符序，其中有一个是默认字符序(Default Collation)；

- `collation`(字符序)名字的规则可以归纳为这两类：
	- `<character set\>_<language/other>_<ci/cs>` (ci表示大小写不敏感，cs表示敏感)
	- `<character set\>_bin`
- 同一种字符集的不同 collate存在的意思：
	- 意义在于在于排序、字符串对比的准确度（相同两个字符在不同国家的语言中的排序规则可能是不同的）以及性能。
###############

mysql使用如下命令来查看字符集与collate(字符序)

	show character set;
	show collation;


### create databases
![](http://i.imgur.com/rl2rWu4.png)

	CREATE database if not EXISTS nimi
	CHARACTER SET utf8
	collate uft8-unicode-ci

### alter database ###
![](http://i.imgur.com/NPumk7r.png)

### create table ###
![](http://i.imgur.com/bpfruLc.png)
![](http://i.imgur.com/kgwL7lC.png)

	create table if not exists world.test(
		ID int(11) not null auto_increment,
		Name char(34) not null default '' comment 'name of this iterm',
		Code char(3) not null default '2',
		District char(23) not null default '2',
		primary key(ID)
		) ENGINE=INNODB auto_increment=2000 default character set=utf8 collate=utf8_UNICODE_ci comment='test table';

格式需要注意的地方：
![](http://i.imgur.com/IDlLw4q.png)


整形字段专用的属性设置有：`UNSIGNED`，`ZEROFILL`(在左侧根据长度的设定填满0)，`AUTO_INCREMENT`
通用字段属性：
- `NOT NULL |　NULL`
- `DEFAULT` 默认值
- `UNIQUE [KEY] | PRIMARY [KEY]`
- `COMMENT` 注释值

还要注意：
- `BLOB` 于 `TEXT`字段不能使用 DEFAULT 关键字来指定默认值

还有关于`TIMESTAMP`字段：
如果你指定了一个字段为TIMESTAMP，系统会默认帮你加上如下属性(如果你没有指定任何属性的话)

	NOT NULL，default CURRENT_TIMESTAMP，on update CURRENT_TIMASTAMP

`CURRENT_TIMESTAMP`代表当前系统的时间戳

`on update CURRENT_TIMASTAMP` 表示当更新当前条记录时MYSQL会自动更新此字段为系统当前时间戳
注意：mysql限制，在一个表格中，CURRENT_TIMESTAMP只能出现在一个字段中

这时候如果我们需要俩字段，一个字段是创建时间，另一个是修改时间，需要同时在俩字段设置`CURRENT_TIMESTAMP`，这时候该怎么办？
![](http://i.imgur.com/Yg2DxNP.png)

### alter table ###
增加字段：

	alter table world.test add column newColumn int;
	alter table world.test add column newColumn2 int FIRST;
	alter table world.test add column newColumn3 char(4) after Code;

通过 [ FIRST | AFTER ... ] 来指定新增栏位的位置

或者通过这样一次性增加多条栏位
	alter table world.test add column (newColumn3 int),(newColumn4 char(4));

### 修改字段 ###
![修改栏位](http://i.imgur.com/k2kimQz.png)

	alter table world.test [change | modify] Code Codes varchar(10) after newColumn3;

### 删除字段 ###

	alter table world.test drop column newColumn3

### 修改表格名称 ###

	alter table world.test rename world.test1; 
	rename table world.test to world.test1

### 删除表格 ###

	drop table [if exists] world.test;

MySql索引

![](http://i.imgur.com/9nmdWAP.png)

INDEX( adress (5) DESC )为address字段建立前5个字符的索引

后期建立索引：

	alter table test add primary key (id)           //create primary Key
	create unique index Name_index on test(Name);   //create unique Key
	create index Codes_District_index on test(Codes,District);

note: create index 不能建立主键索引

### 删除索引 ###

    alter table test
    drop primary key
    drop { INDEX | KEY } 索引名称
	drop index 索引名称 on 表名

note：alter table可以一次性删除多个索引，但是drop index一次只能删除一个索引

LAST_INSERT_ID() 获取上一次插入的自增ID

### 查询表相关信息 ###

	show tables form databaseName [like ""] ;  //查询数据库中的表

MySQL中有一个特别的数据库叫 information_schema(系统资讯数据库)，其中有一个表叫TABLES，里面存放着mysql中表相关的信息

### `show create table` ###

	show create table asset_version64;
返回结果(返回创建这个表格需要的信息)：

    CREATE TABLE `fileInfo` (
      `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
      `inServer` tinyint(1) NOT NULL DEFAULT '1' COMMENT '指定该文件应该是否属于服务器文件，用于打APK包',
      `version` int(10) unsigned NOT NULL,
      `size` int(12) unsigned NOT NULL,
      `md5` char(32) NOT NULL,
      `name` varchar(128) NOT NULL,
      `path` varchar(256) NOT NULL,
      `platformType` char(10) NOT NULL,
      `dependence` blob,
      `timeStamp` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
      `assetNames` blob,
      `isUsed` char(4) NOT NULL DEFAULT '1' COMMENT '是否启动该文件，1 为启动',
      `inPackage` char(4) NOT NULL DEFAULT '0' COMMENT '指定该文件是否在最的APK包内，用于文件的动更',
      `order_name` varchar(128) NOT NULL DEFAULT '0' COMMENT '最终会根据此值分组，其值为name_platformType',
      PRIMARY KEY (`id`),
      KEY `id` (`id`)
    ) ENGINE=MyISAM AUTO_INCREMENT=1071 DEFAULT CHARSET=utf8 COMMENT='文件信息表（存储上传文件信息）' 
####

	[desc | describe] world.test;  //显示表结构
	show columns from world.test;  //显示表字段信息
	show fields from world.test;   //显示表字段信息
####

	show index from fileInfo; //显示索引信息




### 子查询 ###

子查询一般在 `where` 或者 `having` 子句中都可以出现，必须得注意子查询传回的资料时候符合规定，见下图
![](http://i.imgur.com/64uA5Ln.png)

在与运算符比较时，子查询最多只能回传一条记录，而且只能有一个字段的值，如果想回传多条记录可以使用 `IN | NOT IN` 关键字
![](http://i.imgur.com/DTv9MRG.png)


还有关键字 `ALL | ANY` (全部|任何一个)(note: = ANY 与 IN 的效果一样)

	SELECT * from inTable
	where n > [any | all] (
		SELECT n from outTable
	)

现在如果你想查询在亚洲而且政府形式为Public的国家	
![](http://i.imgur.com/0IQJLf4.png)

**一种简单的写法就**
![](http://i.imgur.com/pnOnNaM.png)

将两个或者多个一起放到`where`子句中，然后在一个子查询中返回数据


### 在FROM中使用子查询 ###
![](http://i.imgur.com/SYfoC2r.png)

### 关联子查询 ###


![关联子查询](http://i.imgur.com/nPseJVr.png)
注意关键字 `EXISTS` ，如果子查询有结果返回，那么运算条件判断就是成立的

### 视图 ###
视图是一种虚拟的表格，他并不是一张真实的表，所以它里面根本就不存放数据，你可以将它理解为一个窗口，你可以看见的只是你需求较多的数据，你可以将多个表中你关心的字段放到一个视图上面。然后以后获取数据就非常方便
![视图的建立](http://i.imgur.com/QHFqRB3.png)
然后如果你想操作，查看你建立的视图可以通过 `DESC` ，`alter view` , `drop view`来操作，跟操作一张表差不多




参考链接：

[深入Mysql字符集设置](http://www.laruence.com/2008/01/05/12.html)

[MySQL的Collation和Character set问题](http://www.eamonning.com/blog/view/474)

[慎用MYSQL子查询](http://www.cnblogs.com/zhengyun_ustc/p/slowquery3.html)

[SQL子查询深入理解](http://www.cnblogs.com/CareySon/archive/2011/07/18/2109406.html)

[mysql子查询的弱点](http://hidba.org/?p=260)

[mysql性能优化专题](http://simpleframework.net/news/list?t=MySQL%20%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96%E7%AF%87)