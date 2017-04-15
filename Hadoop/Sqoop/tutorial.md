# Sqoop Tutorial

## Sqoop Import
导入工具将单个表从RDBMS导入HDFS。表中的每一行都被视为HDFS中的一条记录。所有记录作为文本数据存储在文本文件中或作为二进制数据存储在Avro和序列文件中。

`sqoop import (generic-args) (import-args)`

### Example
假设在MySQL数据库服务器中有一个叫userdb的数据库，其下有emp,emp_add和emp_contact三张表。
emp:

id |	name |	deg |	salary |	dept
-- | ------- | ---- | -------- | --------
1201 |	gopal |	manager |	50,000 |	TP
1202 |	manisha |	Proof reader |	50,000 |	TP
1203 |	khalil |	php dev |	30,000 |	AC
1204 |	prasanth |	php dev |	30,000 |	AC
1204 |	kranthi |	admin |	20,000 |	TP

emp_add:

id |	hno |	street |	city
--- | ----- | -------- | --------
1201 |	288A |	vgiri |	jublee
1202 |	108I |	aoc |	sec-bad
1203 |	144Z |	pgutta |	hyd
1204 |	78B |	old city |	sec-bad
1205 |	720X |	hitec |	sec-bad

emp_contact:

id |	phno |	email
--- | ------ | --------
1201 |	2356742 |	gopal@tp.com
1202 | 	1661663 |	manisha@tp.com
1203 |	8887776 |	khalil@ac.com
1204 |	9988774 |	prasanth@ac.com
1205 |	1231231 |	kranthi@tp.com

#### Importing a Table

将emp表从MySQL数据库中导到HDFS
```
$ sqoop import \
--connect jdbc:mysql://localhost/userdb \
--username root \
--table emp --m 1
```

To verify the data in HDFS:

`$ $HADOOP_HOME/bin/hadoop fs -cat /queryresult/part-m-*`

#### Importing into Target Directory

```
$ sqoop import \
--connect jdbc:mysql://localhost/userdb \
--username root \
--table emp_add \
--m 1 \
--target-dir /queryresult
```

#### Import Subset of Table data
我们可以使用Sqoop导入工具中的'where'子句导入表的子集。它在相应的数据库服务器中执行相应的SQL查询，并将结果存储在HDFS中的目标目录中。

```
$ sqoop import \
--connect jdbc:mysql://localhost/userdb \
--username root \
--table emp_add \
--m 1 \
--where "city ='sec-bad'" \
--target-dir /wherequery
```

#### Incremental Import 增量导入
增量导入是一种仅导入表中新添加的行的技术。需要添加“incremental”，“check-column”和“last-value”选项来执行增量导入。

```
$ sqoop import \
--connect jdbc:mysql://localhost/userdb \
--username root \
--table emp \
--m 1 \
--incremental append \
--check-column id \
--last value 1205
```

### Import All Tables
将所有表从RDBMS数据库服务器导入HDFS。每个表数据存储在单独的目录中，目录名称与表名称相同。

`sqoop import-all-tables (generic-args) (import-args)`

Example
```
$ sqoop import-all-tables \
--connect jdbc:mysql://localhost/userdb \
--username root
```
To verify the result:

`$ $HADOOP_HOME/bin/hadoop fs -ls`

## Sqoop Export
导出工具将一组文件从HDFS导出回到RDBMS。目标表必须已存在。作为Sqoop输入提供的文件包含记录，这些记录在表中称为行。它们被读取并解析成一组记录，并用用户指定的分隔符分隔。

`sqoop export (generic-args) (export-args)`

### Example
Data in HDFS:
```
1201, gopal,     manager, 50000, TP
1202, manisha,   preader, 50000, TP
1203, kalil,     php dev, 30000, AC
1204, prasanth,  php dev, 30000, AC
1205, kranthi,   admin,   20000, TP
1206, satish p,  grp des, 20000, GR
```
Create the table _employee_ in mysql:
```
$ mysql
mysql> USE db;
mysql> CREATE TABLE employee ( 
   id INT NOT NULL PRIMARY KEY, 
   name VARCHAR(20), 
   deg VARCHAR(20),
   salary INT,
   dept VARCHAR(10));
```
下面的命令用于将HDFS中的*emp_data*导入到mysql的*employee*表中：
```
$ sqoop export \
--connect jdbc:mysql://localhost/db \
--username root \
--table employee \ 
--export-dir /emp/emp_data
```
To verify the result:

`mysql>select * from employee;`

## Sqoop Job
创建和维护Sqoop作业。 Sqoop作业创建并保存导入和导出命令。它指定用于识别和调用保存的作业的参数。这种重新调用或重新执行在增量导入中使用，可以将更新的行从RDBMS表导入到HDFS。

```
sqoop job (generic-args) (job-args)
   [-- [subtool-name] (subtool-args)]
```

### Create Job --create 
```
$ sqoop job --create myjob \
--import \
--connect jdbc:mysql://localhost/db \
--username root \
--table employee --m 1
```

### Verify Job

`$ sqoop job --list`

### Inspect Job --show
'--show'参数用于检查或验证特定作业及其详细信息。

```
$ sqoop job --show myjob
Job: myjob 
 Tool: import Options:
 ---------------------------- 
 direct.import = true
 codegen.input.delimiters.record = 0
 hdfs.append.dir = false 
 db.table = employee
 ...
 incremental.last.value = 1206
 ...
```

### Execute Job --exec 
用来执行已保存的job。

`$ sqoop job --exec myjob`

## Codegen 
从面向对象的应用程序的角度来看，每个数据库表都有一个DAO类，它包含用于初始化对象的“getter”和“setter”方法。这个工具（-codegen）自动生成DAO类。它在Java中生成DAO类，基于表模式结构。 Java定义作为导入过程的一部分实例化。这个工具的主要用途是检查Java是否丢失了Java代码。如果是这样，它将创建一个新版本的Java，在字段之间使用默认分隔符。

```
$ sqoop codegen \
--connect jdbc:mysql://localhost/userdb \
--username root \ 
--table emp
14/12/23 02:34:40 INFO sqoop.Sqoop: Running Sqoop version: 1.4.5
14/12/23 02:34:41 INFO tool.CodeGenTool: Beginning code generation
……………….
14/12/23 02:34:42 INFO orm.CompilationManager: HADOOP_MAPRED_HOME is /usr/local/hadoop
Note: /tmp/sqoop-hadoop/compile/9a300a1f94899df4a9b10f9935ed9f91/emp.java uses or overrides a deprecated API.
Note: Recompile with -Xlint:deprecation for details.
14/12/23 02:34:47 INFO orm.CompilationManager: Writing jar file: /tmp/sqoop-hadoop/compile/9a300a1f94899df4a9b10f9935ed9f91/emp.jar
```
Verification:
```
$ cd /tmp/sqoop-hadoop/compile/9a300a1f94899df4a9b10f9935ed9f91/
$ ls
emp.class
emp.jar
emp.java
```

## Eval
它允许用户对相应的数据库服务器执行用户定义的查询，并在控制台中预览结果。因此，用户可以期望导入结果表数据。使用eval，我们可以评估任何类型的SQL查询，可以是DDL或DML语句。

`$ sqoop eval (generic-args) (eval-args)`

### Example

#### Select Query Evaluation

```
$ sqoop eval \
--connect jdbc:mysql://localhost/db \
--username root \ 
--query "SELECT * FROM employee LIMIT 3"
+------+--------------+-------------+-------------------+--------+
| Id   | Name         | Designation | Salary            | Dept   |
+------+--------------+-------------+-------------------+--------+
| 1201 | gopal        | manager     | 50000             | TP     |
| 1202 | manisha      | preader     | 50000             | TP     |
| 1203 | khalil       | php dev     | 30000             | AC     |
+------+--------------+-------------+-------------------+--------+
```

#### Insert Query Evaluation
```
$ sqoop eval \
--connect jdbc:mysql://localhost/db \
--username root \ 
-e "INSERT INTO employee VALUES(1207,'Raju','UI dev',15000,'TP')"
```

## List Database

```
$ sqoop list-databases \
--connect jdbc:mysql://localhost/ \
--username root
```

## List Table

```
$ sqoop list-tables \
--connect jdbc://mysql://localhost/userdb \
--username root
```
