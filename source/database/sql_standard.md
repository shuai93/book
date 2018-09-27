
# sql开发中最基本的规范要求
----



## 规范
1. 代码行清晰、整齐、层次分明、结构性强，易于阅读；
2. 代码中应具备必要的注释以增强代码的可读性和可维护性；
3. 代码应充分考虑执行效率，保证代码的高效性；

## 详细sql 总结
1. **查看表字段名或随机少量数据时，不要使用 ` SELECT * FROM TABLENAME `**

 在psql命令窗口用 ` \d TABLENAME ` 或 ` SELECT * FROM TABLENAME WHERE 1 = 2 `、` SELECT * FROM TABLENAME limit 1 `等命令查看表结构信息，尽量不要直接执行 `SELECT * FROM TABLENAME ` ，这样GP数据库会查询出sql所有的记录。

 <br>





3. **查询总记录数时，尽量不要用 ` COUNT(*) `，而要指定一个有索引的字段**

例如主键列为INDEX，使用` COUNT(INDEX) ` 能利用索引。

<br>

4. **对分区表进行查询时，尽量把分区键作为查询条件的第一个条件**

<br>

5. **无条件删除表中数据时，用 ` TRUNCATE ` 代替 ` DELETE `**

<br>

6. **查询语句中尽量使用表的索引字段，避免做大表的全表扫描**

 例如：WHERE子句中有联接的列，即使最后的联接值为一个静态值，也不会使用索引。
 ` SELECT * FROM EMPLOYEE WHERE FIRST_NAME || '' || LAST_NAME = 'Beill Cliton'; `
这条语句没有使用基于LAST_NAME创建的索引。
当采用下面这种SQL语句的编写，GP系统就可以采用基于LAST_NAME创建的索引。
 ` SELECT *  FROM EMPLOYEE WHERE FIRST_NAME = 'Beill' AND LAST_NAME = 'Cliton'; `
<br>
