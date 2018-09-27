
# sql开发中最基本的规范要求
----------

2. **SELECT 子句中避免使用***



 另外在ETL开发过程中使用 ` INSERT INTO..SELECT * FROM.. ` 语句时，如果FROM表新增字段时，会造成代码执行报错可能。
