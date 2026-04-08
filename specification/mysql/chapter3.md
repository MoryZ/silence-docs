# （三）SQL 语句

**1.【强制】`DML`、`DDL`语句中的 SQL 保留字及数据库函数一律大写，表名、列名及别名一律小写。`,`、`=`等符号用空格分隔。**

正例：`SELECT t1.id, t1.name FROM table_first AS t1 , table_second AS t2 WHERE t1.id = t2.id`

反例：`select t1.ID,t1.NAME from TABLE_FIRST as t1,TABLE_SECOND as t2 where t1.id=t2.id`

**2.【强制】不要使用`COUNT(列名)`或`COUNT(常量)`来替代`COUNT(*)`，`COUNT(*)`是`SQL92`定义的标准统计行数的语法，跟数据库无关，跟`NULL`和非`NULL`无关。**

`COUNT(*)`会统计值为`NULL`的行，而`COUNT(列名)`不会统计此列为`NULL`值的行。

**3.【强制】`COUNT(DISTINCT col)`计算该列除`NULL`之外的不重复行数，注意`COUNT(DISTINCT col1, col2)`如果其中一列全为`NULL`，那么即使另一列有不同的值，也返回为 0。**

**4.【强制】当某一列的值全是`NULL`时，`COUNT(col)`的返回结果为0，但`SUM(col)`的返回结果为`NULL`，因此使用`SUM()`时需注意 NPE 问题。**

正例：可以使用如下方式来避免`SUM()`的 NPE 问题：`SELECT IFNULL(SUM(column), 0) FROM table`

**5.【强制】使用`ISNULL()`来判断是否为`NULL`值。**

`NULL`与任何值的直接比较都为`NULL`。

1.  `NULL<>NULL`的返回结果是`NULL`，而不是`false`。

2.  `NULL=NULL`的返回结果是`NULL`，而不是`true`。

3.  `NULL<>1`的返回结果是`NULL`，而不是`true`。

反例：在 SQL 语句中，如果在`NULL`前换行，影响可读性。`SELECT * FROM table WHERE column1 IS NULL AND column3 IS NOT NULL;`而`ISNULL(column)`是一个整体，简洁易懂。从性能数据上分析，`ISNULL(column)`执行效率更快一些。

**6.【强制】代码中写分页查询逻辑时，若`COUNT`为0应直接返回，避免执行后面的分页语句。**

**7.【强制】不得使用外键与级联，一切外键概念必须在应用层解决。**

**8.【强制】禁止使用存储过程，存储过程难以调试和扩展，更没有移植性。**

**9.【强制】数据订正(特别是删除或修改记录操作)时，要先`SELECT`，避免出现误删除，确认无误才能执行更新语句。**

**10.【强制】对于数据库中表记录的查询和变更，只要涉及多个表，都需要在列名前加表的别名（或表名）进行限定。**

对多表进行查询记录、更新记录、删除记录时，如果对操作列没有限定表的别名（或表名），并且操作列在多个表中存在时，就会抛异常。

正例：`SELECT t1.name FROM table_first AS t1, table_second AS t2 WHERE t1.id = t2.id`

反例：在某业务中，由于多表关联查询语句没有加表的别名（或表名）的限制，正常运行两年后，最近在 某个表中增加一个同名字段，在预发布环境做数据库变更后，线上查询语句出现出 1052 异常：`Column 'name' in field list is ambiguous。`

**11.【强制】禁用`UPDATE …​ | DELETE t1 WHERE a = XX LIMIT XX`这种带`LIMIT`的更新语句。**

会导致主从不一致从而引发数据错乱。建议加上`ORDER BY PK`。

**12.【强制】禁止使用关联子查询，如`UPDATE t1 SET name = xxx WHERE name IN (SELECT name FROM user WHERE …​)`，效率极其低下。**

**13.【强制】禁用`INSERT INTO …​ ON DUPLICATE KEY UPDATE …​`，在高并发环境下会造成主从不一致。**

**14.【强制】禁止联表更新语句，如`UPDATE t1, t2 WHERE t1.id = t2.id`。**

**15.【推荐】SQL 语句中表的别名前加`AS`，并且以表的简称或 t1、t2、t3、…​的顺序依次命名。**

1.  别名可以是表的简称，或者是依照表在 SQL 语句中出现的顺序，以 t1、t2、t3 的方式命名。

2.  别名前加 AS 使别名更容易识别。

正例：`SELECT t1.name FROM table_first AS t1 , table_second AS t2 WHERE t1.id = t2.id`

**16.【推荐】`IN`操作能避免则避免，若实在避免不了，需要仔细评估 in 后边的集合元素数量，控制在 1000 个之内。**

**17.【推荐】减少使用`ORDER BY`，和业务沟通能不排序就不排序，或将排序放到程序端去做。`ORDER BY`、`GROUP BY`、`DISTINCT`这些语句较为耗费CPU，数据库的CPU资源是极其宝贵的。**

**18.【推荐】包含了`ORDER BY`、`GROUP BY`、`DISTINCT`这些查询的语句，`WHERE`条件过滤出来的结果集请保持在 1000 行以内，否则 SQL 会很慢。**

**19.【推荐】减少使用`OR`语句，可将`OR`语句优化为`UNION`，然后在各个`WHERE`条件上建立索引。如`WHERE a = 1 OR b = 2`优化为`WHERE a = 1 …​ UNION …​ WHERE b = 2, INDEX(a), INDEX(b)`。**

**20.【参考】因国际化需要，所有的字符存储与表示均采用`utf8`字符集，字符计数方法需要注意。**

`SELECT LENGTH("轻松工作")`返回为 12

`SELECT CHARACTER_LENGTH("轻松工作")`返回为 4

如果需要存储表情，那么选择`utf8mb4`进行存储，注意它与`utf8`编码的区别。

**21.【参考】`TRUNCATE TABLE`比`DELETE`速度快，且使用的系统和事务日志资源少，但`TRUNCATE`无事务且不触发`trigger`，有可能造成事故，故不建议在开发代码中使用此语句。**

`TRUNCATE TABLE`在功能上与不带`WHERE`子句的`DELETE`语句相同。
