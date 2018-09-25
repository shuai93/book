## mysql

### mysql 基本数据

#### 记录常用的sql
- 行转列
  ![source](/assets/source.png)
  ```

  CREATE TABLE `TabName` (
  `Id` int(11) NOT NULL AUTO_INCREMENT,
  `Name` varchar(20) DEFAULT NULL,
  `Date` date DEFAULT NULL,
  `Scount` int(11) DEFAULT NULL,
  PRIMARY KEY (`Id`)
  ) ENGINE=InnoDB  DEFAULT CHARSET=utf8;

  INSERT INTO `TabName` VALUES ('1', '小说', '2013-09-01', '10000');
  INSERT INTO `TabName` VALUES ('2', '微信', '2013-09-01', '20000');
  INSERT INTO `TabName` VALUES ('3', '小说', '2013-09-02', '30000');
  INSERT INTO `TabName` VALUES ('4', '微信', '2013-09-02', '35000');
  INSERT INTO `TabName` VALUES ('5', '小说', '2013-09-03', '31000');
  INSERT INTO `TabName` VALUES ('6', '微信', '2013-09-03', '36000');
  INSERT INTO `TabName` VALUES ('7', '小说', '2013-09-04', '35000');
  INSERT INTO `TabName` VALUES ('8', '微信', '2013-09-04', '38000');

  SELECT Date ,
  MAX(CASE NAME WHEN '小说' THEN Scount ELSE 0 END ) 小说,
  MAX(CASE NAME WHEN '微信' THEN Scount ELSE 0 END ) 微信
  FROM TabName  GROUP BY Date ;

  ```
  ![result](/assets/result.png)


## postgresql
