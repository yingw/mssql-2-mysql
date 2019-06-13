[toc]
# MSSQL 迁移 MYSQL
最近在公司做了mssql迁移到mysql的测试，因为之前存在大量的存储过程，所以对迁移造成了不少的阻力。在此，把遇到的问题做个记录。

## 存储过程及函数写法不同
### 函数

功能|mssql函数 | mysql函数
-----|--|--
添加时间|`DATEADD(datepart,number,date)`：<br/>`date` 参数是合法的日期表达式。<br/>`number` 是您希望添加的间隔数 | `DATE_ADD(date,INTERVAL expr type)`：<br/>`date` 参数是合法的日期表达式。<br/>`expr` 参数是您希望添加的时间间隔。
返回两个日期之间的时间|`DATEDIFF(datepart,startdate,enddate)`：<br/>第二个参数-第一个参数| `DATEDIFF(date1,date2)`：<br/>第一个参数-第二个参数
返回当前的日期和时间|`GETDATE()`|`NOW()`
是否为NULL|`ISNULL()`|`IFNULL()`
用不同的格式显示日期/时间|`CONVERT(data_type(length),data_to_be_converted,style)`：<br/>`data_type(length)` 规定目标数据类型（带有可选的长度）。<br/>`data_to_be_converted` 含有需要转换的值。<br/>`style` 规定日期/时间的输出格式。|`DATE_FORMAT(date,format)`：<br/>`date` 参数是合法的日期。<br/>`format` 规定日期/时间的输出格式。

### 存储过程


语法|mssql | mysql
---|---|---
定义存储过程无参|`CREATE PROCEDURE query`<br>` AS` | `CREATE PROCEDURE query()`
定义存储过程有参|`CREATE PROCEDURE query`<br>` AS`<br>` @a varchar(20)，@b varchar(20) output` | `CREATE PROCEDURE query(IN a VARCHAR(20),OUT b VARCHAR(20))`
执行存储过程|`exec query ‘a’, @b output`|`call query ('a',@b);`
定义变量|`DECLARE @a INT`|`DECLARE a INT DEFAULT 1;`
事务|`BEGIN TRANSACTION`<br>` ...`<br>` COMMIT TRANSACTION`|`START TRANSACTION;`<br>` ... `<br>`COMMIT;`
获取影响行数|`@@ROWCOUNT`|`ROW_COUNT()`
获取事务数|`@@TRANCOUNT`| 无
获取异常|`BEGIN TRY`<br>` ...`<br/>`END TRY`<br/>`BEGIN CATCH`<br>` ... `<br>`END CATCH`|无
IF判断|`IF`<br>`...`<br>`ELSE`|`IF THEN`<br>`...`<br>`ELSE`<br>`...`<br>`END IF`
While 循环|`while 条件`<br>` begin `<br>`执行操作 `<br>` end`|`WHILE 条件 DO`<br>`执行操作`<br>`END WHILE;`

## SQL语法不同

### 原生sql

语法|mssql | mysql
---|---|---
select 赋值|`select a=count(*) ...` |`select count(*) into a ...`
update join set|`update A set A_NAME = B.B_NAME from A left join B ON A.B_ID = B.B_ID`：先set再join|`UPDATE A LEFT JOIN B ON A.B_ID = B.B_ID SET A.A_NAME = B.B_NAME;`：先join再set
字段名|有[],可以是关键字|无[]，最好不要是关键字，如果有加反引号``
表名|有dbo.|无dbo.

### MyBits sql

**亦存在原生sql和存储过程的问题。**

功能|mssql | mysql
---|---|---
分页|`offset(${pagination.rows}*(${pagination.page} - 1)) rows`<br>`fetch next ${pagination.rows} rows only`| `limit ${(pagination.page - 1)*pagination.rows},${pagination.rows};`
模糊查询|`LIKE '%'+#{name}+'%'`| `LIKE CONCAT('%',#{name},'%')`
调用存储过程|`EXEC ab #{a, mode=OUT, jdbcType=INTEGER}, #{b, mode=OUT, jdbcType=INTEGER}`|`{CALL ab (#{a, mode=OUT, jdbcType=INTEGER}, #{b, mode=OUT, jdbcType=INTEGER})}`

**注:mysql结束语句要加;** 以及数据库配置忽略大小写。




