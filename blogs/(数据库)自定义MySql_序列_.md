>MySql只有自增长序列，采用创建函数的方式实现类Oracle序列的效果。

>复制下面的SQL，在MySql上执行即可
```
#第一步：创建--Sequence 管理表
DROP TABLE IF EXISTS sequence; 
CREATE TABLE sequence ( 
     seq_name VARCHAR(50) NOT NULL COMMENT '序列名称,参考格式：EKS$SEQ', 
     current_value BIGINT NOT NULL COMMENT '当前值', 
     increment BIGINT NOT NULL DEFAULT 1 COMMENT '步长(跨度)', 
     PRIMARY KEY (seq_name) 
) ENGINE=InnoDB COMMENT='序列表'; 
 
#第二步：创建--取当前值的函数
DROP FUNCTION IF EXISTS currval; 
DELIMITER $ 
CREATE FUNCTION currval (seq_name_param VARCHAR(50)) 
     RETURNS BIGINT 
     LANGUAGE SQL 
     DETERMINISTIC 
     CONTAINS SQL 
     SQL SECURITY DEFINER 
     COMMENT '' 
BEGIN 
     DECLARE value BIGINT; 
     SET value = 0; 
     SELECT current_value INTO value 
          FROM sequence 
          WHERE seq_name = seq_name_param; 
     RETURN value; 
END 
$ 
DELIMITER ; 
 
#第三步：创建--取下一个值的函数
DROP FUNCTION IF EXISTS nextval; 
DELIMITER $ 
CREATE FUNCTION nextval (seq_name_param VARCHAR(50)) 
     RETURNS BIGINT 
     LANGUAGE SQL 
     DETERMINISTIC 
     CONTAINS SQL 
     SQL SECURITY DEFINER 
     COMMENT '' 
BEGIN 
     UPDATE sequence 
          SET current_value = current_value + increment 
          WHERE seq_name = seq_name_param; 
     RETURN currval(seq_name_param); 
END 
$ 
DELIMITER ; 
 
#第四步：创建--更新当前值的函数
DROP FUNCTION IF EXISTS setval; 
DELIMITER $ 
CREATE FUNCTION setval (seq_name_param VARCHAR(50), value BIGINT) 
     RETURNS BIGINT 
     LANGUAGE SQL 
     DETERMINISTIC 
     CONTAINS SQL 
     SQL SECURITY DEFINER 
     COMMENT '' 
BEGIN 
     UPDATE sequence 
          SET current_value = value 
          WHERE seq_name = seq_name_param; 
     RETURN currval(seq_name_param); 
END 
$ 
DELIMITER ; 
```
```
以下操作可直接使用JDBC操作，SQL语句与下一致。
创建序列：INSERT INTO sequence VALUES ('EKS$SEQ', 0, 1);#添加一个sequence名称和初始值，以及自增幅度
设置序列初始值：SELECT SETVAL('EKS$SEQ', 10);#设置指定sequence的初始值
查询序列当前值：SELECT CURRVAL('EKS$SEQ');#查询指定sequence的当前值
查询序列下一值：SELECT NEXTVAL('EKS$SEQ');#查询指定sequence的下一个值
```

>效果截图：
![image.png](http://upload-images.jianshu.io/upload_images/9541455-da9b4ce5aef55a1d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](http://upload-images.jianshu.io/upload_images/9541455-d9ac135f59495c33.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](http://upload-images.jianshu.io/upload_images/9541455-2291372ef7276003.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](http://upload-images.jianshu.io/upload_images/9541455-a1c4f0f93b3c8f30.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](http://upload-images.jianshu.io/upload_images/9541455-335b3e170f2b9ca5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
参考博客：
在MySQL中创建实现自增的序列（Sequence）的教程
http://www.jb51.net/article/76124.htm
