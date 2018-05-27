# 任意文件下载的另类技巧

## 0x00 前言
-------------------------------
任意文件下载,一般多是在一些web应用提供文件下载的功能,工程师考虑遵循"单一原则",开发一个将请求中的
`file`或`filePath`作为参数，来下载指定的文件。这样开发一个下载的功能，就能支持所有的下载需求了。

比如，输入这样的URL

>>http://xxx.com/download?file=123.txt

就能够下载123.txt这样的文件.这是理所当然的,随着网站安全意识的提高,工程师们为了防止任意文件下载的产
生，但又为了满足那种资源分享类网站的功能,开始对`file` 和`filePath`等这种参数的字段加密,我们经常会看
到下面这种链接

>>http://xxx.com/download?file=c7b8de23fd238fe5e16f6f03b844022f9f72fd168a0704d82d58f19cf72b7aa3

参数的值被加密了,这种该怎么办, 很多时候我们都很绝望遇到这种下载链接,想办法解密这种字段,然而工程师们一旦
加密往往都是hash加盐或者不知道hash了多少次，一般很难还原出加密算法.这就对我们遇到这种链接很绝望的概念.接下来
告诉大家一个有趣的技巧

我们常常可以在网站上可以看到文件下载处右键查看源码是不是类似以下这种的
![](http://image.3001.net/images/20180527/15274058072245.png!small)

或者这种的

![](http://image.3001.net/images/20180527/15274031385788.png)

可能你会注意到图1中的zip后缀的文件名,这样的一般都是前端js把文件名调用后端接口生成一个类似`https://xxx.com/download?flieName=MH005SySetCtrl_downkhdcjload&ID=AFJwgSeGptC9qTzSZHRejs57U3LShoO6`
这种的文件下载链接,那么我们是否可以尝试控制这个文件名生成一个我们想要的文件下载,例如`/etc/password`或`/web/web.config`这种的,如果后端不过滤我们就可以下载我们想要的文件了

## 0x01 尝试构想

在测试网站时尝试抓了抓网站页面加载的请求,找到这个下载文件的函数

``` javascript
function downloadKjxx(fileName){
       var sub =new SwordSubmit();
         sub.pushData("fileName",fileName);  // 文件名称
		 sub.setCtrl('MH0');
		 sub.options.isRedirect="false";
		 sub.options.postType="download"; 
		 sub.submit();
}
```
然后我就在页面的console控制台重新复制了这个函数，调用并下载这个文件 ，处于安全性打码处理

![](https://github.com/LiodAir/images/blob/master/te/aaaaaaa.gif?raw=true)

如图中所示,我们通过更改函数中的fileName为`../../../../../../../../../../../etc/passwd`，成功下载到文件

因此我们可以通过浏览器的console控制台更改函数达到任意文件下载

## 0x03 总结

>通过图中我们更改了js函数中的filename，js会自动生成文件下载链接，但是一般这个真正生成链接的js是加密混淆处理

``` javascript
eval(var a='aaaa'function(){;var d=AddBizCode2URL("/ctrl=WorkFlowCTRL_viewBackReason&workItemId="+c);swordAlertIframe(d,b,null)}}..............此处省略几百行.................);

```
>这样的混淆处理的,当时就放弃了如何查找构造,直接引用就可以下载.

* 文件下载处链接是网页js生成的如`javascript:donload('11.txt')`、`javascript:download('mh00111111')`，这种的，后一种可以看下js
是否能构造可控的filename

* 网页js生成的动态链接是[](http://xxx.com/file=0xaxasxadas2sadsafas0dasd2sadas0dasd0)http://xxx.com/file=0xaxasxadas2sadsafas0dasd2sadas0dasd0 这种的动态下载,而不是静态的`http://xxx.com/xxx.docx`

* 还会遇到目录限制,只能下载web目录的文件

----------------------------------------------------------
`个人原创文章 作者:Lioder`
