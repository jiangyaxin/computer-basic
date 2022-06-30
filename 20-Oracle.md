## 语法

限制行数：ROWNUM < 行数。

分页： `select * from (select * from <table> where ROWNUM < 行数) where ROWNUM > 行数;`

字符串连接：`||`。

函数：`Trim()`,`Upper`,`Length()`,`SubString()`。

`Add_Month()`：给日期加/减月份。

`Extract()`：从日期和时间中提取年、月、日、时、分、秒。

`Last_Day()`：返回月份的最后一天。

`Months_Between()`：返回两个月份之间的月数。

`Next_Day()`：返回指定日期后面一天。

`Sysdate()`：返回当前日期和时间。

`To_Date()`：把字符串转换成日期。

WHERE 和 HAVING 的区别：WHERE 发生在分组之前，HAVING 发生在分组之后。

储存过程：

```sql
CREATE OR REPLACE PROCEDURE <name>(<param-name> <IN|OUT> <type>) IS
BEGAIN
  <sql>
END;
```

游标：

```sql
DECLARE CURSOR <name> IS <select-sql>;
BEGAIN
  OPEN <name>;
  -- 循环取游标
  LOOP
     FETCH <name> INTO <var1>,<var2>,<var3>,<var4>;
         <sql>
     EXIT WHEN <name>%notfound;
  END LOOP;
  ClOSE <name>;
END;
```

SGA共享池：用来缓存SQL语句解析后的内容，叫做库高速缓存、系统参数等，一条SQL不同用户使用也只会解析一次，采用LRU算法管理，可以通过查询 v$sql 视图查看库高速缓存的内容，可以通过 绑定常量 来代替固定常量以使用库高速缓存。

SGA缓冲区缓存：用来缓存从硬盘中读取出来后或写入硬盘之前的数据块，块中的接触计数器表明块的使用频繁程度。

查看执行计划：`EXPLAIN PLAN FOR <sql>`
