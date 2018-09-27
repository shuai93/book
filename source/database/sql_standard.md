
# sql开发中最基本的规范要求




## 规范
1. 代码行清晰、整齐、层次分明、结构性强，易于阅读；
2. 代码中应具备必要的注释以增强代码的可读性和可维护性；
3. 代码应充分考虑执行效率，保证代码的高效性；

## 详细sql 总结

1. **查看表字段名或随机少量数据时，不要使用 ` SELECT * FROM TABLENAME `**

 在psql命令窗口用 ` \d TABLENAME ` 或 ` SELECT * FROM TABLENAME WHERE 1 = 2 `、` SELECT * FROM TABLENAME limit 1 `等命令查看表结构信息，尽量不要直接执行 `SELECT * FROM TABLENAME ` ，这样GP数据库会查询出sql所有的记录。



2. **SELECT 子句中避免使用***


 另外在ETL开发过程中使用 ` INSERT INTO..SELECT * FROM.. ` 语句时，如果FROM表新增字段时，会造成代码执行报错可能。
 所以，直接在SELECT子句中写出想要显示的列。
