# 联合查询

T1表结构

| user_id | user_name | password |
| :-----: | :-------: | :------: |
|    1    |   owen    |  123456  |
|    2    |  miller   |  654321  |

T2表结构

| user_id | score | level |
| :-----: | :---: | :---: |
|    1    |  30   |   5   |
|    2    |  50   |   8   |



### 内联

内联是把两张表中的数据拼成一行。

如果想把一个**user_id**所有的信息(用户名、密码、积分、等级)都查出来，使用内联。

*   SQL：SELECT * FROM T1, T2 WHERE T1.user_id=T2.user_id

还有另一种更高效的写法，一般使用这种：

*   SQL：SELECT * FROM T1 INNER JOIN T2 ON T1.user_id=T2.user_id