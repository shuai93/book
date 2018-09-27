
# sql开发中最基本的规范要求




## 规范
1. 代码行清晰、整齐、层次分明、结构性强，易于阅读；
2. 代码中应具备必要的注释以增强代码的可读性和可维护性；
3. 代码应充分考虑执行效率，保证代码的高效性；

## 详细sql 总结

1. **查看表字段名或随机少量数据时，不要使用 ` SELECT * FROM TABLENAME `**

 在psql命令窗口用 ` \d TABLENAME ` 或 ` SELECT * FROM TABLENAME WHERE 1 = 2 `、` SELECT * FROM TABLENAME limit 1 `等命令查看表结构信息，尽量不要直接执行 `SELECT * FROM TABLENAME ` ，这样GP数据库会查询出sql所有的记录。



2. **SELECT 子句中避免使用***

 在子句中列出所有的列时,使用*很方便，但是效率低。
