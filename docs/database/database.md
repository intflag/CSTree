# 数据库系统原理
## 事务
### 1、概念
事务指的是满足 ACID 特性的一组操作，可以通过 Commit 提交一个事务，也可以使用 Rollback 进行回滚。
![](http://images.intflag.com/database01.png)

### 2、ACID 事务四大特性
**1）原子性（Atomicity）**

事务被视为不可分割的最小单元，事务的所有操作要么全部提交成功，要么全部失败回滚。

回滚可以用回滚日志（Undo Log）来实现，回滚日志记录着事务所执行的修改操作，在回滚时反向执行这些修改操作即可。

**2）一致性（Consistency）**

数据库在事务执行前后都保持一致性状态。在一致性状态下，所有事务对同一个数据的读取结果都是相同的。

**3）隔离性（Isolation）**

一个事务所做的修改在最终提交以前，对其它事务是不可见的。

**4）持久性（Durability）**

一旦事务提交，则其所做的修改将会永远保存到数据库中。即使系统发生崩溃，事务执行的结果也不能丢失。

系统发生奔溃可以用重做日志（Redo Log）进行恢复，从而实现持久性。与回滚日志记录数据的逻辑修改不同，重做日志记录的是数据页的物理修改。


事务的 ACID 特性概念简单，但不是很好理解，主要是因为这几个特性不是一种平级关系：

- 只有满足一致性，事务的执行结果才是正确的。
- 在无并发的情况下，事务串行执行，隔离性一定能够满足。此时只要能满足原子性，就一定能满足一致性。
- 在并发的情况下，多个事务并行执行，事务不仅要满足原子性，还需要满足隔离性，才能满足一致性。
- 事务满足持久化是为了能应对系统崩溃的情况。

### 3、AUTOCOMMIT
默认情况下, MySQL启用自动提交模式（变量 autocommit 为 ON ），这意味着, 只要你执行 DML 操作的语句， MySQL 会立即隐式提交事务（Implicit Commit）。

```
//查看 autocommit 模式
mysql> show session variables like 'autocommit';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| autocommit    | ON    |
+---------------+-------+
1 row in set (0.01 sec)

//修改 autocommit 模式
mysql> set session autocommit=0;
Query OK, 0 rows affected (0.00 sec)

mysql> show session variables like 'autocommit';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| autocommit    | OFF   |
+---------------+-------+
1 row in set (0.02 sec)
```

## SQL 语言四大分类
### 1、DDL 数据定义语言
DDL（Data Definition Language 数据定义语言）用于操作对象和对象的属性，这种对象包括数据库本身，以及数据库对象，如：表、视图等等。

主要操作语句：create、drop、alter
### 2、DML 数据操作语言
DML（Data Manipulation Language 数据操控语言）用于操作数据库对象中包含的数据，也就是说操作的单位是记录。

主要操作语句：insert、delete、update
### 3、DQL 数据查询语言
DQL（Data Query Language 数据查询语言）用于从数据库表中检索数据。

主要操作语句：select
### 4、DCL 数据控制语言
DCL（Data Control Language 数据控制语句）用于控制数据库对象的权限，使数据更加的安全。

主要操作语句：grant、revoke、if…else、while、begin transaction、commit、rollback

注：也有人将事务单独划分为一类，TCL 事务控制语言。