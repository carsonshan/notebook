# Linux命令行操作技巧

本文档不涉及root权限，Linux相关笔记还有：

[Linux系统配置](Linux-setup.md)

[SSH远程登录](Linux-SSH.md)

[Linux备份](Linux-backup.md)

----

# 帮助文本的grep，把stderr重定向到stdout

就是用2>&1这样的重定向咯

    ssh-keygen --help 2>&1|grep bit

----

# 当前目录文件全文搜索

这里要搜索当前目录下面所有的包含"MultiTeam"文件

    grep MultiTeam -r .

----

# 统计当前文件夹代码行数

find 指定文件后缀名，记住要引号避免bash解析*

    find -name "*.py" -o -name "*.md"|xargs cat|wc

----

# 查看给定文件列表的文件大小

用xargs -d指定分隔符为\n（默认会按照空格和\n分隔参数）

```
cat list.txt | xargs -d "\n" ls -alh
```

----

# wget慢慢下载

```
wget -i list.txt  -nc --wait=60 --random-wait
```

其中nc表示已经下载到的文件就不要再请求了，wait=60表示两次请求间隔60s，random-wait表示随机等待2~120s

----

# touch修改时间戳

将b.txt的时间戳改为和a.txt一样

```
touch -r a.txt b.txt
```

----

# 去掉Ubuntu默认情况下ls的颜色

```
unalias ls
```

----

# 换行方式修改

如果一个文件来自于Windows，可能需要先修改换行方式才能用，去掉文件中的\r

vim中输入 `:set ff=unix`

----

# iodine--使用DNS传输数据

>* http://code.kryo.se/iodine/

注意： 本方案网速极低，使用时要有足够的耐心，不能保证复杂情况下是否可行（尤其是Windows）

前期准备：一个域名（假设为example.com）及一台服务器（假设为1.2.3.4），建议客户端在Linux上运行

### 1. 设置域名解析

dns.example.com添加一条A记录，解析至1.2.3.4

t.example.com添加一条NS记录，值为dns.example.com

### 2. 服务器端

    ./iodined -f -c -P secretpassword 192.168.99.1 t.example.com

-f表示持续占用前台，-c表示不限制请求源，-P指定密码，最后是`内网IP`和使用的域名

内网IP可以随意指定，只要当前服务器没有占用即可，例如可以改为172.16.0.1

### 3.检查服务端是否正常

http://code.kryo.se/iodine/check-it/

作者提供了在线检查工具，输入t.example.com即可检查

### 4.客户端

建议在ubuntu等完整的Linux操作系统上运行，下载源码后make即可

     ./iodine -f -P secretpassword t.example.com

效果图：

![](download/img/iodine-finish.jpg)

----

# 远程控制Windows

Windows下有自带的mstsc，Linux如树莓派用啥呢？就用[rdesktop](http://www.rdesktop.org/)啦

手册查询用`man rdesktop`

快速使用：

```
sudo apt-get install -y rdesktop
rdesktop -f -u 用户名 -p 密码 服务器地址:端口
```

其中-f表示全屏

注意上述在命令行中使用明文密码并不安全，可能被其他用户用ps等工具看到，建议仅仅在完全自己控制的Linux上系统上这样操作