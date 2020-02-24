mysql基础部分

## 约束

一种限制，用于限制表中的数据，为了保证表中的数据的准确和可靠性

### 六大约束

- NOT NULL ：非空，用于保证该字段的值不能为空
- DEFAULT：默认，用于保证该字段有默认值
- PRIMARY KEY 主键约束，用于保证该字段的值具有唯一性，并且非空
- UNIQUE：唯一性约束，用于保证该字段的值具有唯一性
- CHECK：检查约束，【mysql中不支持】 比如性别gender
- FOREIGN KEY：外键约束，用于限制俩个表的关系，用于保证该字段的值必须来自主表的关联列的值，再从表添加外键约束。

#### 约束的添加分类

列级约束：六大约束语法上都支持，**但外键约束没有效果**
表级约束：除了非空、默认，其他的都支持

##### 外键约束注意事项

1. 要求在从表中设置外键关系
2. 从表的外键列的类型和主表的关联列的类型要求一致或者是兼容，名称无要求
3. 主表的关联列必须一个key（主键或者唯一性）
4. 插入数据，先插入主表，在插入从表；~~删除数据，先删除从表，再删除主表~~

#### 创建表时约束sql

~~~sql
#添加的主键、唯一、外键约束 表会自动创建相关的索引
#查看stuinfo中的所有索引，包括主键、外键、唯一
SHOW INDEX FROM stuinfo;


#列级约束
USE students;
DROP TABLE stuinfo;
CREATE TABLE stuinfo(
	id INT PRIMARY KEY,#主键
	stuName VARCHAR(20) NOT NULL UNIQUE,#非空
	gender CHAR(1) CHECK(gender='男' OR gender ='女'),#检查
	seat INT UNIQUE,#唯一
	age INT DEFAULT  18,#默认约束
	majorId INT REFERENCES major(id)#外键

);
CREATE TABLE major(
	id INT PRIMARY KEY,
	majorName VARCHAR(20)
);


# 标记约束
DROP TABLE IF EXISTS stuinfo;
CREATE TABLE stuinfo(
	id INT,
	stuname VARCHAR(20),
	gender CHAR(1),
	seat INT,
	age INT,
	majorid INT,
  
	CONSTRAINT pk PRIMARY KEY(id),#主键
	CONSTRAINT uq UNIQUE(seat),#唯一键
	CONSTRAINT ck CHECK(gender ='男' OR gender  = '女'),#检查
	CONSTRAINT fk_stuinfo_major FOREIGN KEY(majorid) REFERENCES major(id)#外键
);


#通用写法
CREATE TABLE IF NOT EXISTS stuinfo(
	id INT PRIMARY KEY,
	stuname VARCHAR(20),
	sex CHAR(1),
	age INT DEFAULT 18,
	seat INT UNIQUE,
	majorid INT,
	CONSTRAINT fk_stuinfo_major FOREIGN KEY(majorid) REFERENCES major(id)
);
~~~

#### 修改表时添加约束sql

1、添加列级约束
alter table 表名 modify column 字段名 字段类型 新约束;

2、添加表级约束
alter table 表名 add 【constraint 约束名】 约束类型(字段名) 【外键的引用】;

~~~sql
DROP TABLE IF EXISTS stuinfo;
CREATE TABLE stuinfo(
	id INT,
	stuname VARCHAR(20),
	gender CHAR(1),
	seat INT,
	age INT,
	majorid INT
)
DESC stuinfo;

#1.添加非空约束
ALTER TABLE stuinfo MODIFY COLUMN stuname VARCHAR(20)  NOT NULL;

#2.添加默认约束
ALTER TABLE stuinfo MODIFY COLUMN age INT DEFAULT 18;

#3.添加主键
#①列级约束
ALTER TABLE stuinfo MODIFY COLUMN id INT PRIMARY KEY;
#②表级约束
ALTER TABLE stuinfo ADD PRIMARY KEY(id);

#4.添加唯一
#①列级约束
ALTER TABLE stuinfo MODIFY COLUMN seat INT UNIQUE;
#②表级约束
ALTER TABLE stuinfo ADD UNIQUE(seat);

#5.添加外键
ALTER TABLE stuinfo ADD CONSTRAINT fk_stuinfo_major FOREIGN KEY(majorid) REFERENCES major(id); 
~~~

#### 修改表删除约束sql

~~~sql
#1.删除非空约束
ALTER TABLE stuinfo MODIFY COLUMN stuname VARCHAR(20) NULL;

#2.删除默认约束
ALTER TABLE stuinfo MODIFY COLUMN age INT ;

#3.删除主键
ALTER TABLE stuinfo DROP PRIMARY KEY;

#4.删除唯一
ALTER TABLE stuinfo DROP INDEX seat;

#5.删除外键
ALTER TABLE stuinfo DROP FOREIGN KEY fk_stuinfo_major;

SHOW INDEX FROM stuinfo;
~~~



## 自增列（标识列）

可以不用手动的插入值，系统提供默认的序列值

### 特点

1. 标识列必须和主键搭配吗？不一定，但**要求是一个key**
2. 一个表可以有几个标识列？至多一个！
3. 标识列的类型只能是数值型
4. 标识列可以通过 SET auto_increment_increment=3 设置起始值;也可以通过手动插入值，设置起始值

### SQL

~~~sql
# 创建表时
DROP TABLE IF EXISTS tab_identity;
CREATE TABLE tab_identity(
	id  int(11) PRIMARY KEY AUTO_INCREMENT,
	name VARCHAR(20)
)AUTO_INCREMENT = 1000;

#修改表时
ALTER table tab_identity AUTO_INCREMENT = 1024;


#修改整个库的主键开始值
SHOW VARIABLES LIKE '%auto_increment%';
#自增的起始值
set auto_increment_offset = 1024;   
#每次增加的数值
SET auto_increment_increment=1;
~~~



## 事物

一个或者一组SQL语句组成的一个执行单元，这个执行方法要么全部执行成功，要么全部不执行

在mysql中用的最多的存储引擎有：innodb，myisam ,memory 等。其中innodb支持事务，而myisam、memory等不支持事务

~~~sql
#查看SQL存储引擎。
show engines
~~~

### 事物的特性（ACID）

- 原子性（Atomicity）： 原子性是指事物是一个不可分割的工作单位，事物中的操作要么发生，要么不发生。
- 一致性（Consistency）：事物必须让数据库从一个一致性状态变化为另一个一致性状态。
- 隔离性（Isolation）：一个事物的执行不能被其他的事物干扰，即一个的事物内部的操作及使用的数据对并发的其他事物是隔离的，并发执行的各个事物之间不能相互干扰
- 持久性（Durability）：一个事物一旦被提交，他对数据库中的数据改变是永久性的，接下来其他的操作和数据库故障不应该对其有任何影响

### 并发事物问题

对于俩个事物A、B

- **脏读**：A读取了已经被B更新但还没有被提交的字段，若B发生回滚，A读取的内容就是临时无效的
- **不可重复的读**：在一个事务A中多次操作一个数据，在这两次或多次访问这个数据的中间，事务B也操作此数据，并使其值发生了改变，这就导致同一个事务A在两次操作这个数据的时候值不一样，这就是不可重复读。）。
- **幻读**：是指事务不独立执行产生的一种现象。事务A读取与搜索条件相匹配的若干行。事务B以插入或删除行等方式来修改事务A的结果集，然后再提交。这样就会导致当A本来执行的结果包含B执行的结果，这两个本来是不相关的，对于A来说就相当于产生了“幻觉”。

### 数据库的事物隔离级别

一个事务与其他事务隔离的程度称为隔离级别. 数据库规定了多种事务隔离级别, 不同隔离级别对应不同的干扰程度, 隔离级别越高, 数据一致性就越好, 但并发性越弱

#### 数据库提供的4中隔离级别

| 隔离级别                         | 描述                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| READ UNCOMMITTED（读未提交数据） | 允许事物读取未被提交事物提交的更改，脏读、不可重复读，幻读都会出现 |
| READ COMMITTED                   | 只允许事物读取已经被其他事物提交的变更，可以避免脏读，但不可重复读和幻读的问题依然存在 |
| REPEATABLE READ                  | 确保事物可以多次从一个字段中读取相同的值，在这个事物持续期间，精致其他事物对这个字段进行修改，可以避免脏读和不可重复读，但是幻读依然存在 |
| SERIALIZABLE                     | 确保事物可以从一个表中读取相同的行，在这个事物持续期间，禁止其他事物对该表执行插入，更新和删除操作，所有的并发问题都可以避免，但是性能低下 |

- Oracle 支持的 2 种事务隔离级别：**READ COMMITED**, SERIALIZABLE。 Oracle 默认的事务隔离级别为: READ COMMITED 
- Mysql 支持 4 种事务隔离级别. Mysql 默认的事务隔离级别为: **REPEATABLE READ**
- **每启动一个 mysql 程序, 就会获得一个单独的数据库连接. 每个数据库连接都有一个全局变量 @@tx_isolation, 表示当前的事务隔离级别**

#### 事物隔离级别sql

~~~sql
#查看当前的隔离级别:
SELECT @@tx_isolation;

#设置当前 mySQL 连接的隔离级别:
set transaction isolation level read committed

#设置数据库系统的全局的隔离级别：
set global transaction isolation level read committed
~~~

#### 开启数据库事物的sql

~~~sql
步骤1：开启事务
set autocommit=0;
start transaction;可选的
步骤2：编写事务中的sql语句(select insert update delete)
语句1;
语句2;
...

步骤3：结束事务
commit;提交事务
rollback;回滚事务

SHOW VARIABLES LIKE 'autocommit';
SHOW ENGINES;

#1.演示事务的使用步骤
#开启事务
SET autocommit=0;
START TRANSACTION;
#编写一组事务的语句
UPDATE account SET balance = 1000 WHERE username='张无忌';
UPDATE account SET balance = 1000 WHERE username='赵敏';
#结束事务
ROLLBACK;
#commit;
SELECT * FROM account;

#2.演示事务对于delete和truncate的处理的区别
SET autocommit=0;
START TRANSACTION;
DELETE FROM account;
ROLLBACK;

#3.演示savepoint 的使用
SET autocommit=0;
START TRANSACTION;
DELETE FROM account WHERE id=25;
SAVEPOINT a;#设置保存点
DELETE FROM account WHERE id=28;
ROLLBACK TO a;#回滚到保存点

SELECT * FROM account;
~~~

## 视图

虚拟表和普通表一样使用，mysql5.1版本出现的新特性，是通过表动态生成的数据，**只是保存了sql逻辑	增删改查，只是一般不能增删改**

### 创建视图sql

~~~sql
#创建视图
语法：
create view 视图名
as
查询语句;

#案例：查询姓张的学生名和专业名
SELECT stuname,majorname
FROM stuinfo s
INNER JOIN major m ON s.`majorid`= m.`id`
WHERE s.`stuname` LIKE '张%';

#创建视图sql
CREATE VIEW v1
AS
SELECT stuname,majorname
FROM stuinfo s
INNER JOIN major m ON s.`majorid`= m.`id`;
#查询视图
SELECT * FROM v1 WHERE stuname LIKE '张%';
~~~

### 修改视图sql

~~~sql

SELECT * FROM myv3 
#方式一
语法：
create or replace view  视图名
as
查询语句;

CREATE OR REPLACE VIEW myv3
AS
SELECT AVG(salary),job_id
FROM employees
GROUP BY job_id;

#方式二
语法：
alter view 视图名
as 
查询语句;

ALTER VIEW myv3
AS
SELECT * FROM employees;

~~~

### 删除视图sql

~~~sql
语法：drop view 视图名,视图名,...;
DROP VIEW emp_v1,emp_v2,myv3;
~~~

### 查看视图sql

~~~sql
DESC myv3;

SHOW CREATE VIEW myv3;
~~~

### 更新视图sql

~~~sql
CREATE OR REPLACE VIEW myv1
AS
SELECT last_name,email
FROM employees;

SELECT * FROM myv1;
SELECT * FROM employees;
#1.插入
INSERT INTO myv1 VALUES('张飞','zf@qq.com');

#2.修改
UPDATE myv1 SET last_name = '张无忌' WHERE last_name='张飞';

#3.删除
DELETE FROM myv1 WHERE last_name = '张无忌';
~~~

#### 具备以下特点的视图不允许更新

- **包含以下关键字的sql语句：分组函数（sum() 求和，avg() 求平均值，max() 求最大值，min() 求最小值，count() 计算个数）、distinct、group  by、having、union或者union all**

~~~sql
CREATE OR REPLACE VIEW myv1
AS
SELECT MAX(salary) m,department_id
FROM employees
GROUP BY department_id;

SELECT * FROM myv1;

#更新
UPDATE myv1 SET m=9000 WHERE department_id=10;

#②常量视图
CREATE OR REPLACE VIEW myv2
AS

SELECT 'john' NAME;

SELECT * FROM myv2;

#更新
UPDATE myv2 SET NAME='lucy';
~~~

- **常量视图**

~~~sql
CREATE OR REPLACE VIEW myv2
AS

SELECT 'john' NAME;

SELECT * FROM myv2;

#更新
UPDATE myv2 SET NAME='lucy';
~~~

- **Select中包含子查询**

~~~sql
CREATE OR REPLACE VIEW myv3
AS

SELECT department_id,(SELECT MAX(salary) FROM employees) 最高工资
FROM departments;

#更新
SELECT * FROM myv3;
UPDATE myv3 SET 最高工资=100000;
~~~

- **join**

~~~sql
CREATE OR REPLACE VIEW myv4
AS

SELECT last_name,department_name
FROM employees e
JOIN departments d
ON e.department_id  = d.department_id;

#更新
SELECT * FROM myv4;
UPDATE myv4 SET last_name  = '张飞' WHERE last_name='Whalen';
INSERT INTO myv4 VALUES('陈真','xxxx');
~~~

- **from一个不能更新的视图**

~~~sql
CREATE OR REPLACE VIEW myv5
AS

SELECT * FROM myv3;

#更新

SELECT * FROM myv5;

UPDATE myv5 SET 最高工资=10000 WHERE department_id=60;
~~~

- **where子句的子查询引用了from子句中的表**

~~~sql
CREATE OR REPLACE VIEW myv6
AS

SELECT last_name,email,salary
FROM employees
WHERE employee_id IN(
	SELECT  manager_id
	FROM employees
	WHERE manager_id IS NOT NULL
);

#更新
SELECT * FROM myv6;
UPDATE myv6 SET salary=10000 WHERE last_name = 'k_ing';
~~~

## 变量

### 系统变量

变量由系统定义，不是用户定义，属于服务器层次

#### 语法

1、查看所有系统变量
show global|【session】variables;
2、查看满足条件的部分系统变量
show global|【session】 variables like '%char%';
3、查看指定的系统变量的值
select @@global|【session】系统变量名;
4、为某个系统变量赋值
方式一：
set global|【session】系统变量名=值;
方式二：
set @@global|【session】系统变量名=值;

注

1. **全局变量需要添加global关键字，会话变量需要添加session关键字，如果不写，默认会话级别**
2. **全局变量作用域：针对于所有会话（连接）有效，但不能跨重启**
3. **会话变量作用域：针对于当前会话（连接）有效**

#### 全局变量sql

~~~sql
#①查看所有全局变量
SHOW GLOBAL VARIABLES;
#②查看满足条件的部分系统变量
SHOW GLOBAL VARIABLES LIKE '%char%';
#③查看指定的系统变量的值
SELECT @@global.autocommit;
#④为某个系统变量赋值
SET @@global.autocommit=0;
SET GLOBAL autocommit=0;
~~~

#### 会话变量sql

~~~sql
#①查看所有会话变量
SHOW SESSION VARIABLES;
#②查看满足条件的部分会话变量
SHOW SESSION VARIABLES LIKE '%char%';
#③查看指定的会话变量的值
SELECT @@autocommit;
SELECT @@session.tx_isolation;
#④为某个会话变量赋值
SET @@session.tx_isolation='read-uncommitted';
SET SESSION tx_isolation='read-committed';
~~~

### 自定义变量

变量用户自定义，系统不提供

使用步骤：
1、声明
2、赋值
3、使用（查看、比较、运算等）

#### 用户变量sql

**作用域：针对于当前会话（连接）有效，作用域同于会话变量**

~~~sql
#声明并初始化（= 或:=）
set @变量名=值;
set @变量名:=值;
select @变量名:=值;

#赋值（更新查操作）
#方式一
set @变量名=值;
set @变量名:=值;
select @变量名:=值;
#方式二
select 字段 into @变量名 from 表

#使用
select @变量名

#案例：声明两个变量，求和并打印

#用户变量
SET @m=1;
SET @n=1;
SET @sum=@m+@n;
SELECT @sum;
~~~

#### 局部变量sql

**作用域：仅仅在定义它的begin end块中有效应用在 begin end中的第一句话**

~~~sql
#声明
declare 变量名 类型;
declare 变量名 类型 default 值;

#赋值
#方式一
set 局部变量名=值;
set 局部变量名:=值;
select 局部变量名:=值;
#方式二
select 字段 into 局部变量名 from 表;

#使用
select 局部变量名;

declare abc int(11);
declare abcd int(11) default 0;

set abc = 1;
set abc:= 2;
select abc:=3;
select id into abc from user;

select abc;
~~~

#### 用户变量和局部变量的对比

|          | 作用域                | 定义位置            | 语法                    |
| -------- | --------------------- | ------------------- | ----------------------- |
| 用户变量 | 当前会话              | 会话的任何地方      | 加@符号，不用定义类型   |
| 局部变量 | 定义在他的BEGIN END中 | BEGIN END的第一句话 | 一般不加@符号，定义类型 |

## 存储过程和函数

类似于java中的方法，提高代码的 重用行，简化操作

区别：

**存储过程：可以有0个返回，也可以有多个返回，适合做批量插入、批量更新**
**函数：有且仅有1 个返回，适合做处理数据后返回一个结果**

### 存储过程

语句预先编译好的SQL语句集合，理解成成批处理语句

1. 提高代码的重用性
2. 简化操作
3. 减少了编译次数并且减少了和数据库服务的连接次数，提高了效率

#### 语法

~~~sql
crate procedure 存储过程(参数列表)
DEGIN
#存储过程体(一组合法的SQL语句)
END

#调用
call 存储过程名(实参列表);
~~~

注：

1. 参数列表包含三个部分
   参数模式 参数名 参数类型
   in       stuname     varchar(20)
   参数模式
   in：该参数可以作为输入，也就是该参数需要调用方传入值
   out：该参数可以作为输出，也就是该参数可以作为返回值
   inout：该参数既可以作为输入又可以作为输出，也就是该参数既需要传入值，又可以返回值

2. 如果存储过程体仅仅只有一句话，begin end可以省略
3. 存储过程体中的每条sql语句的结尾要求必须加分号。
4. 存储过程的结尾可以使用 delimiter 重新设置
   语法：
   delimiter 结束标记
   案例：
   delimiter $

#### sql

~~~sql
#1.空参列表
#案例：插入到admin表中五条记录

SELECT * FROM admin;
#一般使用;即可
DELIMITER $
CREATE PROCEDURE myp1()
BEGIN
	INSERT INTO admin(username,`password`) 
	VALUES('john1','0000'),('lily','0000'),('rose','0000'),('jack','0000'),('tom','0000');
END $


#调用
CALL myp1()$

#2.创建带in模式参数的存储过程

#案例1：创建存储过程实现 根据女神名，查询对应的男神信息

CREATE PROCEDURE myp2(IN beautyName VARCHAR(20))
BEGIN
	SELECT bo.*
	FROM boys bo
	RIGHT JOIN beauty b ON bo.id = b.boyfriend_id
	WHERE b.name=beautyName;
	

END $

#调用
CALL myp2('柳岩')$

#案例2 ：创建存储过程实现，用户是否登录成功

CREATE PROCEDURE myp4(IN username VARCHAR(20),IN PASSWORD VARCHAR(20))
BEGIN
	DECLARE result INT DEFAULT 0;#声明并初始化
	
	SELECT COUNT(*) INTO result#赋值
	FROM admin
	WHERE admin.username = username
	AND admin.password = PASSWORD;
	
	SELECT IF(result>0,'成功','失败');#使用
END $

#调用
CALL myp3('张飞','8888')$


#3.创建out 模式参数的存储过程
#案例1：根据输入的女神名，返回对应的男神名

CREATE PROCEDURE myp6(IN beautyName VARCHAR(20),OUT boyName VARCHAR(20))
BEGIN
	SELECT bo.boyname INTO boyname
	FROM boys bo
	RIGHT JOIN
	beauty b ON b.boyfriend_id = bo.id
	WHERE b.name=beautyName ;
	
END $


#案例2：根据输入的女神名，返回对应的男神名和魅力值

CREATE PROCEDURE myp7(IN beautyName VARCHAR(20),OUT boyName VARCHAR(20),OUT usercp INT) 
BEGIN
	SELECT boys.boyname ,boys.usercp INTO boyname,usercp
	FROM boys 
	RIGHT JOIN
	beauty b ON b.boyfriend_id = boys.id
	WHERE b.name=beautyName ;
	
END $


#调用
CALL myp7('小昭',@name,@cp)$
SELECT @name,@cp$



#4.创建带inout模式参数的存储过程
#案例1：传入a和b两个值，最终a和b都翻倍并返回

CREATE PROCEDURE myp8(INOUT a INT ,INOUT b INT)
BEGIN
	SET a=a*2;
	SET b=b*2;
END $

#调用
SET @m=10$
SET @n=20$
CALL myp8(@m,@n)$
SELECT @m,@n$


#三、删除存储过程
#语法：drop procedure 存储过程名
DROP PROCEDURE p1;
DROP PROCEDURE p2,p3;#×

#四、查看存储过程的信息
DESC myp2;×
SHOW CREATE PROCEDURE  myp2;
~~~

### 函数

含义：一组预先编译好的SQL语句的集合，理解成批处理语句
1、提高代码的重用性
2、简化操作
3、减少了编译次数并且减少了和数据库服务器的连接次数，提高了效率

### 语法

~~~SQL
#一、创建语法
CREATE FUNCTION 函数名(参数列表) RETURNS 返回类型
BEGIN
	函数体
END

#二、调用语法
SELECT 函数名(参数列表)
~~~

注意：

1. 参数列表 包含两部分：
参数名 参数类型

2. 函数体：肯定会有return语句，如果没有会报错
  如果return语句没有放在函数体的最后也不报错，但不建议

  return 值;

3. 函数体中仅有一句话，则可以省略begin end

4. 使用 delimiter语句设置结束标记

#### sql

~~~sql
#1.无参有返回
#案例：返回公司的员工个数
CREATE FUNCTION myf1() RETURNS INT
BEGIN

	DECLARE c INT DEFAULT 0;#定义局部变量
	SELECT COUNT(*) INTO c#赋值
	FROM employees;
	RETURN c;
	
END $

SELECT myf1()$


#2.有参有返回
#案例1：根据员工名，返回它的工资
CREATE FUNCTION myf2(empName VARCHAR(20)) RETURNS DOUBLE
BEGIN
	SET @sal=0;#定义用户变量 
	SELECT salary INTO @sal   #赋值
	FROM employees
	WHERE last_name = empName;
	
	RETURN @sal;
END $

SELECT myf2('k_ing') $

#案例2：根据部门名，返回该部门的平均工资
CREATE FUNCTION myf3(deptName VARCHAR(20)) RETURNS DOUBLE
BEGIN
	DECLARE sal DOUBLE ;
	SELECT AVG(salary) INTO sal
	FROM employees e
	JOIN departments d ON e.department_id = d.department_id
	WHERE d.department_name=deptName;
	RETURN sal;
END $

SELECT myf3('IT')$

#三、查看函数
SHOW CREATE FUNCTION myf3;

#四、删除函数
DROP FUNCTION myf3;

#案例
#一、创建函数，实现传入两个float，返回二者之和
CREATE FUNCTION test_fun1(num1 FLOAT,num2 FLOAT) RETURNS FLOAT
BEGIN
	DECLARE SUM FLOAT DEFAULT 0;
	SET SUM=num1+num2;
	RETURN SUM;
END $

SELECT test_fun1(1,2)$
~~~



## 控制流程（顺序、分支、循环）

### 分支结构

#### 条件函数

##### if函数

语法：if(条件,值1，值2)
功能：实现双分支
应用在begin end中或外面

##### case函数

情况一: 类似于switch函数

~~~sql
case 变量或表达式
when 值1 then 结果1
when 值2 then 结果2
...
else 结果n
end 
~~~

情况二：

~~~SQL
case 
when 条件1 then 结果1
when 条件2 then 结果2
...
else 结果n
end 
~~~

#### 分支结构

##### if结构

~~~sql
语法：
if 条件1 then 语句1;
elseif 条件2 then 语句2;
....
else 语句n;
end if;
功能：类似于多重if

只能应用在begin end 中

#案例1：创建函数，实现传入成绩，如果成绩>90,返回A，如果成绩>80,返回B，如果成绩>60,返回C，否则返回D
CREATE FUNCTION test_if(score FLOAT) RETURNS CHAR
BEGIN
	DECLARE ch CHAR DEFAULT 'A';
	IF score>90 THEN SET ch='A';
	ELSEIF score>80 THEN SET ch='B';
	ELSEIF score>60 THEN SET ch='C';
	ELSE SET ch='D';
	END IF;
	RETURN ch;
		
END $

SELECT test_if(87)$


#案例2：创建存储过程，如果工资<2000,则删除，如果5000>工资>2000,则涨工资1000，否则涨工资500
CREATE PROCEDURE test_if_pro(IN sal DOUBLE)
BEGIN
	IF sal<2000 THEN DELETE FROM employees WHERE employees.salary=sal;
	ELSEIF sal>=2000 AND sal<5000 THEN UPDATE employees SET salary=salary+1000 WHERE employees.`salary`=sal;
	ELSE UPDATE employees SET salary=salary+500 WHERE employees.`salary`=sal;
	END IF;
	
END $

CALL test_if_pro(2100)$
~~~

##### case结构

情况一：类似于switch

~~~SQL
case 变量或表达式
when 值1 then 语句1;
when 值2 then 语句2;
...
else 语句n;
end CASE;

DROP FUNCTION if EXISTS test_case2;
CREATE FUNCTION test_case2(score int) RETURNS CHAR
BEGIN 
DECLARE result CHAR DEFAULT 'A';
case score 
when 90 then set result = 'A';
when 80 then set result = 'B';
when 70 then set result = 'C';
else set result ='D';
end CASE;
RETURN result;
END;
select test_case2(80);
~~~

情况二:

~~~sql 
case 
when 条件1 then 语句1;
when 条件2 then 语句2;
...
else 语句n;
end CASE;

应用在begin end 中或外面

#案例1：创建函数，实现传入成绩，如果成绩>90,返回A，如果成绩>80,返回B，如果成绩>60,返回C，否则返回D
CREATE FUNCTION test_case(score FLOAT) RETURNS CHAR
BEGIN 
	DECLARE ch CHAR DEFAULT 'A';
	CASE 
	WHEN score>90 THEN SET ch='A';
	WHEN score>80 THEN SET ch='B';
	WHEN score>60 THEN SET ch='C';
	ELSE SET ch='D';
	END CASE;
	RETURN ch;
END;

SELECT test_case(56);
~~~

### 循环结构

分类：
while、loop、repeat

循环控制：

iterate类似于 continue，继续，结束本次循环，继续下一次
leave 类似于  break，跳出，结束当前所在的循环

#### while

~~~SQL
语法：
[标签:] while 循环条件 do
 循环体;
 end while 标签;

类似java中
while(循环条件){
	循环体;
}


#1.没有添加循环控制语句
/*
联想
int i=1;
while(i<=insertcount){

	//插入
	
	i++;

}
*/
#案例：批量插入，根据次数插入到admin表中多条记录
DROP PROCEDURE pro_while1$
CREATE PROCEDURE pro_while1(IN insertCount INT)
BEGIN
	DECLARE i INT DEFAULT 1;
	WHILE i<=insertCount DO
		INSERT INTO admin(username,`password`) VALUES(CONCAT('Rose',i),'666');
		SET i=i+1;
	END WHILE;
	
END ;
CALL pro_while1(100);

#2.添加leave语句
#案例：批量插入，根据次数插入到admin表中多条记录，如果次数>20则停止
TRUNCATE TABLE admin$
DROP PROCEDURE test_while1$
CREATE PROCEDURE test_while1(IN insertCount INT)
BEGIN
	DECLARE i INT DEFAULT 1;
	a:WHILE i<=insertCount DO
		INSERT INTO admin(username,`password`) VALUES(CONCAT('xiaohua',i),'0000');
		IF i>=20 THEN LEAVE a;
		END IF;
		SET i=i+1;
	END WHILE a;
END $
CALL test_while1(100)$

#3.添加iterate语句
/*
联想
int i=0;
while(i<=insertCount){
	i++;
	if(i%2==0){
		continue;
	}
	插入
	
}
*/
#案例：批量插入，根据次数插入到admin表中多条记录，只插入偶数次
TRUNCATE TABLE admin$
DROP PROCEDURE test_while1$
CREATE PROCEDURE test_while1(IN insertCount INT)
BEGIN
	DECLARE i INT DEFAULT 0;
	a:WHILE i<=insertCount DO
		SET i=i+1;
		IF MOD(i,2)!=0 THEN ITERATE a;
		END IF;
		
		INSERT INTO admin(username,`password`) VALUES(CONCAT('xiaohua',i),'0000');
		
	END WHILE a;
END $
CALL test_while1(100)$
~~~

#### loop

~~~SQL
语法：
【标签:】loop
	循环体;
end loop 【标签】;

可以用来模拟简单的死循环
~~~

#### repeat

~~~SQL
语法：
【标签：】repeat
	循环体;
until 结束循环的条件
end repeat 【标签】;

联想：
do{

}while();
~~~

