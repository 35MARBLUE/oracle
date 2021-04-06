***姓名：杨辰露
学号：201810414105
班级：18软工1班***

# 实验三

## 实验目的
掌握分区表的创建方法，掌握各种分区方式的使用场景。
## 实验内容
* 本实验使用3个表空间：USERS,USERS02,USERS03。在表空间中创建两张表：订单表(orders)与订单详表(order_details)。
* 使用你自己的账号创建本实验的表，表创建在上述3个分区，自定义分区策略。
* 你需要使用system用户给你自己的账号分配上述分区的使用权限。你需要使用system用户给你的用户分配可以查询执行计划的权限。
* 表创建成功后，插入数据，数据能并平均分布到各个分区。每个表的数据都应该大于1万行，对表进行联合查询。
* 写出插入数据的语句和查询数据的语句，并分析语句的执行计划。
* 进行分区与不分区的对比实验。

## 实验步骤


通过查询可以知道,表空间目前没有USERS02与USERS03,只有USERS，所以需要先创建USERS02，USERS03。
```sql
    SELECT TABLESPACE_NAME,STATUS,CONTENTS,LOGGING 
    FROM dba_tablespaces;
```

![](0.png)

### 步骤1：
创建表空间USERS02，USERS03：

#### sql语句
```sql
    --创建USERS02
    CREATE TABLESPACE USERS02 DATAFILE
    '/home/oracle/app/oracle/oradata/orcl/pdborcl/pdbtest_user02_1.dbf'
    SIZE 100M
    AUTOEXTEND ON
    NEXT 50M
    MAXSIZE UNLIMITED,
    '/home/oracle/app/oracle/oradata/orcl/pdborcl/pdbtest_user02_2.dbf'
    SIZE 100M
    AUTOEXTEND ON
    NEXT 50M
    MAXSIZE UNLIMITED
    EXTENT MANAGEMENT LOCAL SEGMENT SPACE MANAGEMENT AUTO;

    --创建USERS03
    CREATE TABLESPACE USERS03 DATAFILE
    '/home/oracle/app/oracle/oradata/orcl/pdborcl/pdbtest_user03_1.dbf'
    SIZE 100M
    AUTOEXTEND ON
    NEXT 50M
    MAXSIZE UNLIMITED,
    '/home/oracle/app/oracle/oradata/orcl/pdborcl/pdbtest_user02_2.dbf'
    SIZE 100M
    AUTOEXTEND ON
    NEXT 50M
    MAXSIZE UNLIMITED
    EXTENT MANAGEMENT LOCAL SEGMENT SPACE MANAGEMENT AUTO;
```

#### 执行结果：
![](1-1.png)
![](1-2.png)

### 步骤2：
首先创建自己的账号new_user_ycl，然后以system身份登录:：
```sql
    [student@deep02 ~]$sqlplus system/123@localhost/pdborcl
    SQL>ALTER USER new_user_ycl QUOTA UNLIMITED ON USERS;
    SQL>ALTER USER new_user_ycl QUOTA UNLIMITED ON USERS02;
    SQL>ALTER USER new_user_ycl QUOTA UNLIMITED ON USERS03;
    SQL>exit
```


#### 执行结果：
![](2-1.png)
![](2-2.png)


### 步骤3：
用新建的账号new_user_ycl登录,并运行脚本文件test3.sql:

```sql
    [student@deep02 ~]$cat test3.sql
    [student@deep02 ~]$sqlplus new_user_ycl/123@localhost/pdborcl
    SQL>@test3.sql
    SQL>exit
```



#### 执行结果
![](3-1.png)
![](3-2.png)



### 步骤4：
用以下样例查看表空间的数据库文件，并且查看每个文件的磁盘占用情况：
```sql
    $ sqlplus system/123@202.115.82.8/czm
    SQL> SELECT tablespace_name,FILE_NAME,BYTES/1024/1024 MB,MAXBYTES/1024/1024 MAX_MB,autoextensible FROM dba_data_files  WHERE  tablespace_name='USERS';
    SQL>SELECT a.tablespace_name "表空间名",Total/1024/1024 "大小MB",
    free/1024/1024 "剩余MB",( total - free )/1024/1024 "使用MB",
    Round(( total - free )/ total,4)* 100 "使用率%"
    from (SELECT tablespace_name,Sum(bytes)free
            FROM   dba_free_space group  BY tablespace_name)a,
        (SELECT tablespace_name,Sum(bytes)total FROM dba_data_files
            group  BY tablespace_name)b
    where  a.tablespace_name = b.tablespace_name;
```
### 查询结果
![](4-1.png)
- autoextensible是显示表空间中的数据文件是否自动增加。
- MAX_MB是指数据文件的最大容量


### 步骤5：
可以在SQL-DEVELOPER上面登陆账号查看数据：Oracle地址：202.115.82.8 用户名：system,hr,用户名 ， 密码123， 数据库名称：pdborcl/或者你的数据库，端口号：1521

```sql
    SQL> SELECT tablespace_name,FILE_NAME,BYTES/1024/1024 MB,MAXBYTES/1024/1024 MAX_MB,autoextensible FROM dba_data_files  WHERE  tablespace_name='USERS';
```
### 查询结果
![](5-1.png)
![](5-2.png)




### 分析
本次实验我们首先学习了数据库对用户权限进行管理，之后学习了如何实现实验分区表的创建、从表以及主表的创建,和讲数据存入不同的分区等相关操作。再之后，我们还通过分析查询语句的执行，对于海量数据存储的表，进行分区后可以更加快速的查询所需要的信息。另外，对表进行分区后，我们可以在多分区存储数据，这样如果表的某个分区出现故障，表在其他分区的数据也仍然可用。最后从执行结果图中可以查看到表空间的数据库文件，以及每个文件的磁盘占用情况。
