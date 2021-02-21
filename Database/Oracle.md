# Oracle

## 命令

`sqlplus`运行程序后立刻登录。

`sqlplus /nolog`只运行程序不立即登录。

`sqlplus sys/123456 as sysdba`以管理员身份登录。

`sqlplus <user_name>/<user_password> @[hostname:port/]<service_name>`登录到某个服务，只能进入数据库容器，不能直接进入可插拔式数据库。

`conn[ect] sys/123456 as sysdba`以管理员身份登录。

`conn[ect] <user_name>/<user_password> @[hostname:port/]<service_name>`登录到某个服务。

`show user`显示当前用户。

`show con_name`显示当前容器名。

`show pdbs`显示所有可插拔式数据库。

`select [column] from v$serives;`查询所有服务的视图。

`select [column] from v$instance;`查询所有实例的视图。

`select [column] from v$pdbs;`查询所有可插拔式数据库的视图。

`select con_id, dbid, guid, name , open_mode from v$pdbs;`

`select sys_context ('USERENV', 'CON_NAME') from dual;`查看当前使用容器：当前是pdb还是cdb。

`select cdb from v$database;`查看数据库是否为多租户数据库（CDB）。

`select * from v$dbfile;`查看PDB的存放位置。

`create pluggable database <pdb_name> admin user <admin_name> identified by <admin_password> file_name_convert=('pdbseed','orclpdbv2');`使用种子pdbseed(同级目录下)创建，相对路径，pdbseed指向存储位置orclpdbv2。

`select tablespace_name from dba_tablespaces;`查询当前用户所有表空间，必须以`SYSDBA`的身份登录。

`select username, default_tablespace from user_users;`查看当前用户的默认表空间。

`drop tablespace <tablespace_name> including contents;`如果存在表空间就删除。

```sql
--创建表空间
create TABLESPACE <tablespace_name>
   LOGGING datafile 'E:\zs\sunway\oracle\orclpdbv2.dbf'
   size 500m
   autoextend on
   next 500m maxsize unlimited
   extent management local;
```

`create user <user_name> identified by <user_password> default tablespace <tablespace_name> temporary tablespace <temp_tablespace_name> quota 20m on users;`创建用户（如果创建的pdb已经创建了用户，则该步骤可以省略）。

`select * from user_role_privs;`查看当前用户角色。

`select * from user_sys_privs;`查看当前用户的系统权限和表级权限。

```sql
-- 普通用户scott默认未解锁，不能进行那个使用，新建的用户也没有任何权限，bai必须授予权限
-- 管理员授权
grant create session to zhangsan; -- 授予zhangsan用户创建session的权限，即登陆权限
grant unlimited session to zhangsan; -- 授予zhangsan用户使用表空间的权限
grant create table to zhangsan; -- 授予创建表的权限
grante drop table to zhangsan; -- 授予删除表的权限
grant insert table to zhangsan; -- 插入表的权限
grant update table to zhangsan; -- 修改表的权限
grant all to public; -- 这条比较重要，授予所有权限(all)给所有用户(public)
-- oralce对权限管理比较严谨，普通用户之间也是默认不能互相访问的，需要互相授权
grant select on tablename to zhangsan; -- 授予zhangsan用户查看指定表的权限
grant drop on tablename to zhangsan; -- 授予删除表的权限
grant insert on tablename to zhangsan; -- 授予插入的权限
grant update on tablename to zhangsan; -- 授予修改表的权限
grant insert(id) on tablename to zhangsan;
grant update(id) on tablename to zhangsan; -- 授予对指定表特定字段的插入和修改权限，注意，只能是insert和update
grant alert all table to zhangsan; -- 授予zhangsan用户alert任意表的权限
```

`alter pluggable database <pdb_name> open|close;`打开/关闭某个可插拔式数据库。

`alter session set container=CDB$ROOT;`切换回数据库容器。

`alter session set container=<container_name>;`切换到某个容器。

`alter user <user_name> identified by <user_password>;`更改某个用户的密码。

`lsnrctl`可以控制监听器。

## CDB 与 PDB

![CDB与PDB的关系图](https://img2018.cnblogs.com/blog/1459131/201812/1459131-20181226105008433-131589082.png)

[oracle_新增pdb实例](https://blog.csdn.net/zs_life/article/details/101266871)

## 启动与关闭

[ORA-01033 ORACLE 正在初始化或关闭](https://www.cnblogs.com/kongkedewoniu-921929/p/4575882.html)

## 连接 Spring Boot

### 两种连接模式：

```properties
// thin模式
spring.datasource.url=jdbc:oracle:thin:@hostname:port/<service_name>
// oci模式
spring.datasource.url=jdbc:oracle:oci:@<service_name>
```

[Java连接Oracle两种方式thin与oci区别](https://blog.csdn.net/kevin_pso/article/details/54949476)：

- 从使用上来说，oci必须在客户机上安装oracle客户端或才能连接，而thin就不需要，因此从使用上来讲thin还是更加方便，这也是thin比较常见的原因。 
- 原理上来看，thin是纯java实现tcp/ip的c/s通讯；而oci方式,客户端通过native java method调用c library访问服务端，而这个c library就是oci(oracle called interface)，因此这个oci总是需要随着oracle客户端安装（从oracle10.1.0开始，单独提供OCI Instant Client，不用再完整的安装client）。
- 它们分别是不同的驱动类别，oci是二类驱动， thin是四类驱动，但它们在功能上并无差异。 

### SID 与 Service

一个数据库对应一个服务（Service），一个数据库对应多个实例（SID）。服务名（service_name）的缺省值为：<database_name>.<database_domain>，例如：orcl.microdone.cn。

```properties
// 监听SID服务
spring.datasource.url=jdbc:oracle:thin:@<hostname>:<port>:<sid_name>
// 监听服务名
spring.datasource.url=jdbc:oracle:thin:@<hostname>:<port>/<service_name>
// 监听TNSName
spring.datasource.url=jdbc:oracle:thin:@<tnsname>
```

[ORACLE中SID和SERVICE_NAME的区别](https://blog.csdn.net/zhangzl1012/article/details/50752572)

## 基本语句

### 选择 SELECT

### 排序 ORDER BY

请注意，ORDER BY子句总是SELECT语句中的最后一个子句。优先级：column_1 > column_2 > column_3 > ...

```sql
SELECT
    column_1,
    column_2,
    column_3,
    ...
FROM
    table_name
ORDER BY
    column_1 [ASC | DESC] [NULLS FIRST | NULLS LAST],
    column_2 [ASC | DESC] [NULLS FIRST | NULLS LAST],
    column_3 [ASC | DESC] [NULLS FIRST | NULLS LAST],
    ...
```

### 去重 DISTINCT

### 过滤 WHERE

### AND 和 OR

### FETCH

```sql
[ OFFSET offset ROWS ]
  FETCH  NEXT [  row_count | percent PERCENT  ] ROWS [ ONLY | WITH TIES ]
```



### 游标

[Oracle 游标详解（cursor）](https://blog.csdn.net/qq_34745941/article/details/81294166)

### 连接 JOIN

#### 交叉连接 CROSS JOIN

```sql
-- 如果不带WHERE条件子句，它将会返回被连接的两个表的笛卡尔积，返回结果的行数等于两个表行数的乘积
-- 举例,下列A、B、C 执行结果相同，但是效率不一样：
SELECT * FROM table1 CROSS JOIN table2 -- A
SELECT * FROM table1, table2 -- B
SELECT * FROM table1 a inner JOIN table2 b -- C

SELECT a.*, b.* FROM table1 a, table2 b WHERE a.id = b.id -- A
SELECT * FROM table1 a cross JOIN table2 b WHERE a.id = b.id -- B(注：cross join后加条件只能用where，不能用on)
SELECT * FROM table1 a inner JOIN table2 b ON a.id = b.id -- C
-- 一般不建议使用方法A和B，因为如果有WHERE子句的话，往往会先生成两个表行数乘积的行的数据表然后才根据WHERE条件从中选择
-- 因此，如果两个需要求交际的表太大，将会非常非常慢，不建议使用
```

#### 内连接 INNER JOIN

```sql
-- 两边表同时符合条件的组合

-- 如果仅仅使用
SELECT * FROM table1 INNER JOIN table2
/*
    内连接如果没有指定连接条件的话，和笛卡尔积的交叉连接结果一样，但是不同于笛卡尔积的地方是，没有笛卡尔积那么复杂要先生成行数乘积的数据表，内连接的效率要高于笛卡尔积的交叉连接。
    但是通常情况下，使用INNER JOIN需要指定连接条件。
*/

/*
    关于等值连接和自然连接
    等值连接(=号应用于连接条件, 不会去除重复的列)
    自然连接(会去除重复的列)
    数据库的连接运算都是自然连接，因为不允许有重复的行（元组）存在。
*/
SELECT * FROM table1 AS a INNER JOIN table2 AS b ON a.column = b.column
```

#### 外连接 OUTER JOIN

```sql
-- 指定条件的内连接，仅仅返回符合连接条件的条目。
/*
    外连接则不同，返回的结果不仅包含符合连接条件的行，而且包括左表(左外连接时), 右表(右连接时)或者两边连接(全外连接时)的所有数据行。
    1)左外连接LEFT [OUTER] JOIN
    显示符合条件的数据行，同时显示左边数据表不符合条件的数据行，右边没有对应的条目显示NULL
*/
SELECT * FROM table1 AS a LEFT [OUTER] JOIN table2 AS b ON a.column = b.column

/*
    2)右外连接RIGHT [OUTER] JOIN
    显示符合条件的数据行，同时显示右边数据表不符合条件的数据行，左边没有对应的条目显示NULL
*/
SELECT * FROM table1 AS a RIGHT [OUTER] JOIN table2 AS b ON a.column = b.column

/*
	3)全外连接full [outer] join
	显示符合条件的数据行，同时显示左右不符合条件的数据行，相应的左右两边显示NULL，即显示左连接、右连接和内连接的并集
*/
SELECT * FROM table1 AS a FULL [OUTER] JOIN table2 AS b ON a.column = b.column
```

#### 自连接 SELF JOIN

### 约束 CONSTRAINT

#### 主键 PRIMARY KEY

```sql
-- 建表时声明
CREATE TABLE table_name (
  column1 datatype null/not null,
  column2 datatype null/not null,
  ...

  CONSTRAINT pk_name
    PRIMARY KEY (column1, column2, ... column_n)
);
-- 修改
ALTER TABLE table_name
ADD CONSTRAINT pk_name
  PRIMARY KEY (column1, column2, ... column_n);
-- 删除、启用、禁用
ALTER TABLE table_name
{ DROP | ENABLE | DISABLE }  CONSTRAINT pk_name
```

#### 外键 FOREIGN KEY

```sql
-- 建表时声明
CREATE TABLE table_name (
  column1 datatype null/not null,
  column2 datatype null/not null,
  ...

  CONSTRAINT fk_name
    FOREIGN KEY (column1, column2, ... column_n)
    REFERENCES parent_table (column1, column2, ... column_n)
);
-- 修改
ALTER TABLE table_name
ADD CONSTRAINT fk_name
  FOREIGN KEY (column1, column2, ... column_n)
  REFERENCES parent_table (column1, column2, ... column_n);
-- 删除、启用、禁用
ALTER TABLE table_name
{ DROP | ENABLE | DISABLE } CONSTRAINT fk_name;
```

### DUAL 表

#### 序列 SEQUENCE

Navicat 中查询序列时，得到的是两倍自增的结果。

```sql
CREATE SEQUENCE sequence_name
    [MAXVALUE num|NOMAXVALUE]
    [MINVALUE num|NOMINVALUE]
    [START WITH num]
    [INCREMENT BY increment]
    [CYCLE|NOCYCLE]
    [CACHE num|NOCACHE]
```

### 物化视图

```sql
create materialized view view_name
refresh [fast|complete|force]
[
on [commit|demand] |
start with (start_time) next (next_time)
]
AS select 查询语句;
```

### 行转列

```sql
SELECT * FROM （数据查询集）
PIVOT 
(
 SUM(Score/*行转列后 列的值*/) FOR 
 coursename/*需要行转列的列*/ IN (转换后列的值)
)
```

### 列转行

```sql
select 字段 from 数据集
unpivot（自定义列名/*列的值*/ for 自定义列名 in（列名））
```

### 索引

[Oracle pctfree,pctused,initrans,maxtrans](https://www.cnblogs.com/HiJacky/p/5492181.html)

```sql
create [unique]|[bitmap] index index_name -- UNIQUE表示唯一索引、BITMAP位图索引
on table_name (column1,column2...|[express]) -- express表示函数索引
  [tablespace tab_name] -- 表示索引存储的表空间
  [pctfree n1] -- 索引块的空闲空间，保留的空间留给更新该块数据使用，对于倾向于查询的应用系统，或倾向于查询的表格，设置为1左右就已经足够
  [initrans n2] -- 初始化事务槽的个数，大部分情况下没有必要调整，1-4足够用了
  [maxtrans n3] -- 控制最大并发事务（可淘汰）
  [storage ( -- 存储块的空间
    initial 64K -- 初始64k
    next 1M
    minextents 1
    maxextents unlimited
  )]
  [NOLOGGING] --表示创建和重建索引时允许对表做DML操作，默认情况下不应该使用
  [NOLINE]
  [NOSORT]; --表示创建索引时不进行排序，默认不适用，如果数据已经是按照该索引顺序排列的可以使用
```

```sql
alter index index_name rename { to index_newname | coalesce | rebuild };
```

```sql
drop index index_name;
```

