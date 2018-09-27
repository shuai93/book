
# sql开发中最基本的规范要求




## 规范
1. 代码行清晰、整齐、层次分明、结构性强，易于阅读；
2. 代码中应具备必要的注释以增强代码的可读性和可维护性；
3. 代码应充分考虑执行效率，保证代码的高效性；

## 详细sql 总结

1. **查看表字段名或随机少量数据时，不要使用 ` SELECT * FROM TABLENAME `**

 在psql命令窗口用 ` \d TABLENAME ` 或 ` SELECT * FROM TABLENAME WHERE 1 = 2 `、` SELECT * FROM TABLENAME limit 1 `等命令查看表结构信息，尽量不要直接执行 `SELECT * FROM TABLENAME ` ，这样GP数据库会查询出sql所有的记录。



2. **SELECT 子句中避免使用 \***

 在 SELECT 子句中列出所有的列时，使用*很方便，但是效率低。因为 GP 在解析过程中会查询数据字典，将 \* 依次转换成所有的列名。另外在ETL开发过程中使用 ` INSERT INTO..SELECT * FROM.. ` 语句时，如果FROM表新增字段时，会造成代码执行报错可能。所以，直接在SELECT子句中写出想要显示的列。



3. **查询总记录数时，尽量不要用 ` COUNT(*) `，而要指定一个有索引的字段**

例如主键列为INDEX，使用` COUNT(INDEX) ` 能利用索引。



4. **对分区表进行查询时，尽量把分区键作为查询条件的第一个条件**



5. **无条件删除表中数据时，用 ` TRUNCATE ` 代替 ` DELETE `**



6. **查询语句中尽量使用表的索引字段，避免做大表的全表扫描**

 例如：WHERE子句中有联接的列，即使最后的联接值为一个静态值，也不会使用索引。
 ` SELECT * FROM EMPLOYEE WHERE FIRST_NAME || '' || LAST_NAME = 'Beill Cliton'; `
这条语句没有使用基于LAST_NAME创建的索引。
当采用下面这种SQL语句的编写，GP系统就可以采用基于LAST_NAME创建的索引。
 ` SELECT *  FROM EMPLOYEE WHERE FIRST_NAME = 'Beill' AND LAST_NAME = 'Cliton'; `


7. **带通配符（%）的LIKE语句**

 例如：SQL语句：
 ` SELECT * FROM EMPLOYEE WHERE LAST_NAME LIKE '%cliton%'; `
 由于通配符（%）在词首出现，所以GP系统不使用LAST_NAME的索引。
 当通配符出现在字符串其他位置时，优化器就能利用索引。例如：
  ` SELECT * FROM EMPLOYEE WHERE LAST_NAME LIKE 'c%'; `



8. **用EXISTS替代IN**

 在许多基于基础表的查询中，为了满足一个条件，往往需要对另一个表进行联接。在这种情况下，使用EXISTS(或NOT EXISTS)通常将提高查询的效率。
```
-- 低效：
SELECT * FROM EMP
WHERE EMPNO > 0  
AND DEPTNO IN (SELECT DEPTNO FROM DEPT WHERE LOC = 'MELB');

-- 高效：
SELECT *
FROM EMP
WHERE EMPNO > 0
	AND EXISTS (
		SELECT 'X'
		FROM DEPT
		WHERE DEPT.DEPTNO = EMP.DEPTNO
			AND LOC = 'MELB'
	);
```
9. **尽可能用UNION ALL替换UNION**

当SQL语句需要UNION两个查询结果集合时，这两个结果集合会以UNION ALL的方式被合并， 然后在输出最终结果前进行排序。如果用UNION ALL替代UNION，就不需排序，提高了查询效率。例如：

```sql
-- 低效：SELECT ACCT_NUM, BALANCE_AMT
  FROM DEBIT_TRANSACTIONS
 WHERE TRAN_DATE = '31-DEC-95'
UNION
SELECT ACCT_NUM, BALANCE_AMT
  FROM DEBIT_TRANSACTIONS
 WHERE TRAN_DATE = '31-DEC-95';

-- 高效：
SELECT ACCT_NUM, BALANCE_AMT
  FROM DEBIT_TRANSACTIONS
 WHERE TRAN_DATE = '31-DEC-95'
UNION ALL
SELECT ACCT_NUM, BALANCE_AMT
  FROM DEBIT_TRANSACTIONS
 WHERE TRAN_DATE = '31-DEC-95';
```
10. **ORDER BY语句建议**

ORDER BY语句对要排序的列没有特别限制，也可以将函数加入列中。在ORDER BY语句中使用非索引项或有计算表达式都将降低查询速度。当ORDER BY中所有的列定义为非空时会用到索引，例如：T1表的ID列存在索引，且非空。则以下查询用到索引： ` SELECT * FROM T1 ORDER BY ID; `



11. **避免使用NOT**

在查询时经常在WHERE子句使用一些逻辑表达式，如大于、小于、等于以及不等于等等，也可以使用AND（与）、OR（或）以及NOT（非）。NOT可用来对任何逻辑运算符号取反。下面是一个NOT子句的例子：` ... WHERE NOT (STATUS ='VALID') `

如果要使用NOT，则应在取反的短语前面加上括号，并在短语前面加上NOT运算符。NOT运算符包含在另外一个逻辑运算符中，这就是不等于（<>）运算符。换句话说，即使不在查询WHERE子句中显式地加入NOT词，NOT仍在运算符中，见下例：` ... WHERE STATUS <>'INVALID'; `
再看下面这个例子：` SELECT * FROM EMPLOYEE WHERE SALARY <> 3000; `
对这个查询，可以改写为不使用NOT：` SELECT * FROM EMPLOYEE WHERE SALARY < 3000 OR SALARY > 3000; `
虽然这两种查询的结果一样，但是第二种查询方案会比第一种查询方案更快些。第二种查询对SALARY列使用索引，而第一种查询则不能使用索引。



12. **使用DECODE函数减少处理时间**

使用DECODE函数可以避免重复扫描相同记录或重复连接相同的表。例如：
```sql
SELECT COUNT(*) ，SUM(SAL)
  FROM EMP
 WHERE DEPT_NO = 0020
   AND ENAME LIKE　 'SMITH%';

SELECT COUNT(*) ，SUM(SAL)
  FROM EMP
 WHERE DEPT_NO = 0030
   AND ENAME LIKE　 'SMITH%';
```
可以用DECODE函数高效地得到相同结果

```sql
SELECT COUNT(DECODE(DEPT_NO, 0020, 'X', NULL)) D0020_COUNT,
       COUNT(DECODE(DEPT_NO, 0030, 'X', NULL)) D0030_COUNT,
       SUM(DECODE(DEPT_NO, 0020, SAL, NULL)) D0020_SAL,
       SUM(DECODE(DEPT_NO, 0030, SAL, NULL)) D0030_SAL
FROM EMP
WHERE ENAME LIKE 'SMITH%';
```
类似的，DECODE函数也可以运用于GROUP BY 和ORDER BY子句中。



13. **如果可以使用WHERE条件，尽量不要在HAVING中限制数据**


14. **尽量不要使用distinct语句对数据去重**

 DISTINCT语句会引起排序操作，当查询的数据量很大时将排序会十分消耗数据库资源，从而影响查询的效率。请使用group by 语句替代distinct语句。



15. **避免在索引列上使用计算**

WHERE子句中，如果索引列是函数的一部分，优化器将不使用索引而使用全表扫描。例如：
```sql
-- 低效：
SELECT * FROM DEPT WHERE SAL * 12 > 25000;

-- 高效：
SELECT * FROM DEPT WHERE SAL > 25000 / 12;
```


16. **避免在索引列上使用IS NULL和IS NOT NULL**

避免在索引中使用任何可以为空的列，GP将无法使用该索引。对于单列索引，如果列包含空值，索引中将不存在此记录。对于复合索引，如果每个列都为空，索引中同样不存在此记录。如果至少有一个列不为空，则记录存在于索引中。

例如：如果唯一性索引建立在表的A列和B列上，并且表中存在一条记录的A，B值为(123，null) ， GP将不接受下一条具有相同A，B值（123，null）的记录(插入)。然而如果所有的索引列都为空，GP将认为整个键值为空，而空不等于空。因此可以无限条空记录。

因空值不存在于索引列中，所以WHERE子句中对索引列进行空值比较将使GP停用该索引。例如：
```
-- 低效：(索引失效)
SELECT … FROM DEPARTMENT WHERE DEPT_CODE IS NOT NULL;

-- 高效：(索引有效)
SELECT … FROM DEPARTMENT WHERE DEPT_CODE >=0;
```



17. **子查询改写成表连接**

通常来说，采用表连接的方式比子查询更有效率，但并不是所有的子查询都可以改写成表连接的形式。只有当连接字段存在唯一性时才可以进行改写。否则重复字段会产生笛卡尔积。
```sql
例如：
T1表存在以下数据：
ID        ID2
1          1
2          2
2          3
T2表存在以下数据：
ID        ID2
1          1
2          2
3          2
4          2
2          3

-- 则以下查询结果不同：
-- 子查询：
SELECT COUNT(*) FROM T1 WHERE T1.ID IN (SELECT ID FROM T2);  --返回值3
-- 表连接：
SELECT COUNT(*) FROM T1, T2 WHERE T1.ID = T2.ID; -- 返回值5
```


18. **使用索引的第一个列**

如果索引是建立在多个列上，只有在它的第一个列(leading column)被WHERE子句引用时，优化器才会选择使用该索引。


19. **减少对表的查询在含有子查询的SQL语句中，要特别注意减少对表的查询**

例如：
```sql
-- 低效：

SELECT TAB_NAME FROM TABLES
WHERE TAB_NAME = (SELECT TAB_NAME FROM TAB_COLUMNS WHERE VERSION = 604)
AND DB_VER = (SELECT DB_VER FROM TAB_COLUMNS WHERE VERSION = 604);

-- 高效：
SELECT TAB_NAME FROM TABLES
WHERE (TAB_NAME, DB_VER) =
 (SELECT TAB_NAME, DB_VER FROM TAB_COLUMNS WHERE VERSION = 604);
```



20. **SQL语句中：用>=替代>**

如果在ID列上建有索引，则语句 ` SELECT * FROM EMPLOYEE WHERE ID >= 9 ` 要比语句 ` SELECT * FROM EMPLOYEE WHERE ID > 8` 高效。这是由于前者DBMS将直接跳到第一个ID等于9的记录而后者将首先定位到8的记录并且向前扫描到第一个DEPT大于9的记录。



21. **大批量数据导入**

大批量数据插入操作：`insert into xx select ..from xx` 操作时，可以使用gp外部表导出在导入。通常ETL工具会提供这样的封装功能来实现批量数据提交。



22. **SQL嵌套层数不能超过两层**

 SQL嵌套层数过多会影响最优执行计划的生成，执行计划容易变化，最终造成SQL执行效率降低，影响数据库的稳定性。因此规范SQL嵌套层数不能超过两层。


23. **定期使用Vacuum analyze tablename 回收垃圾和收集统计信息**



24. **SQL执行计划**

在提交大的查询之前，首先使用Vacuum analyze tablename收集表统计信息，然后使用explain分析执行计划、发现潜在优化机会，避免将系统资源熬尽。



25. **gphdfs外部表**

创建gphdfs外部表时注意尽量使用text格式，所以需提前在hadoop定义表时注意表的存储格式。发现hadoop中Parquet格式的表在GP中使用gphdfs外部表访问时，一旦其文件个数过多（大概20几个）会报错，严重会引起数据库异常。



26. **表增加分区注意事项**

GP数据库增加分区时（ETL代码也要注意），如果需要压缩，则一定要指定压缩属性，如下例子所示:（新增分区不会自动继承父表的压缩属性）
```sql
alter table table_xxx  add partition m201604 VALUES('201604')
WITH (appendonly=true, compresslevel=5, orientation=column, compresstype=zlib)
 --行存储
WITH (appendonly=true, compresslevel=5, orientation=row, compresstype=zlib)

```



27. **delete语句使用规范**

如果可以使用truncate语句替代delete语句，则必须使用truncate语句（比如清空整张表，清空某个分区）。
Delete语句执行完后，选择合理的时机对该表进行vacuum full操作：`vacuum full table_xxx`此操作可以收回delete操作后留下的空闲空间，防止delete频繁操作导致表膨胀。
