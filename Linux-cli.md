# Linux命令行操作技巧

本文档一般不涉及root权限，Linux相关笔记还有：

[Linux系统配置](Linux-setup.md)

[SSH远程登录](Linux-SSH.md)

[Linux备份](Linux-backup.md)

----

# 查看内置命令的帮助

将以下内容加入`~/.bashrc`中即可，判断如果在内置命令就调用help -m，不是则绕开bash函数来运行man进程

```sh
man () {
    case "$(type -t -- "$1")" in
    builtin|keyword)
        help -m "$1" | sensible-pager
        ;;
    *)
        command man "$@"
        ;;
    esac
}
```

----

# grep搜索帮助文档

用两个横线`--`作为grep的第一个参数表示不要把其后面的形如`-z`的参数当成grep的参数

例如我想知道tar命令中的-z是什么意思：

```bash
man tar|grep -- -z
```

# 帮助文本的grep，把stderr重定向到stdout

某些时候帮助文本是输出到标准错误输出的，需要用2>&1这样的重定向咯

    ssh-keygen --help 2>&1|grep bit

----

# 各种解压命令

tar.gz： `tar -zxvf xx.tar.gz`

tar.bz2： `tar -jxvf xx.tar.bz2`

zip：`unzip xx.zip`

参数含义：

-x解压，-v详细显示解压出来的东西（如果是一个复杂的压缩包建议不要用以加快解压速度），-f后接压缩文件的文件名

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

----

# 统计以特定字符串开头的文件数目

awk是个很好用的工具呢，支持substr函数，用法为substr(源字符串，开始，长度)，其中开始从1计数

`ls -l` 长列表显示的话，按空格分就是$9

```
ls -l|awk '{if(substr($9,1,字符串长度)=="你要的那个字符串") print $9}'|sort|uniq|wc -l
```

----

# hexdump查看字符内部编码

echo的-n参数表示不要末尾加\n

```
echo -n hello | hexdump -C
```

----

# 子目录大小排序

sort的-h表示按人类理解的大小格式排序，-r表示逆序

```
du -sh * | sort -hr
```

----

# 安装ffmpeg

在ubuntu14下是没有ffmpeg的官方包支持的，需要添加mc3man的ppa

```
sudo add-apt-repository ppa:mc3man/trusty-media
#按回车继续
sudo apt-get update
sudo apt-get install -y ffmpeg
```

----

# 保证脚本安全执行set -ex

`set`命令挺有用的呢，-e表示如果后面的语句返回不为0立刻结束shell，-x表示显示出每条命令及参数

从[人家的Dockerfile](https://github.com/Medicean/VulApps/blob/master/s/struts2/s2-032/Dockerfile)中学习得来

----

# change readonly bash variable

bash is a weird thing...

declaring a variable as reference by using `declare -n`, we can change it!

```
$ a=1
$ readonly a
$ a=2
bash: a: readonly variable
#Look here!
$ declare -n a
$ a=2
$ echo $a
2
```

----

# 永久等待 sleep infinity

有时写了一个sh文件后需要保持这个sh的运行，就用sleep永久等待好咯

```
sleep infinity
```

----

# zmap扫描整个网段特定开放端口

zmap的运行需要root权限，用`apt-get install zmap`即可安装

更详细的帮助去看看`zmap --help`咯

```
#需要先编辑黑名单 vi /etc/zmap/blacklist.conf 取消掉注释
zmap 192.168.0.0/16 -B1000M -i eth0 -g -T 4  -p 23 -o 23.txt
```

其中`-g`表示扫描结束后显示总结，`-T 4`表示启动4个扫描线程，`-p 23`表示扫描23端口，-o保存文件的名称

如果拨号了vpn，需要用-G指定网关的MAC地址，可以通过`arp 网关的IP`得到


----

# 对ip列表批量测试redis未授权漏洞

```
for i in `cat iplist.txt`; do (if [ `echo PING|redis-cli -h $i` == "PONG" ] ;then echo $i;fi);done 2>/dev/null
```

利用了bash支持的for语句，注意for之后的分号和最后的done

还有用了if字符串相等，记得要用fi结束if

redis-cli连接上服务器后发送PING，如果存在未授权访问漏洞则会返回PONG，否则会要求Auth或者其他报错信息

----

# 使用ImageMagick对图像进行裁剪

安装命令：`sudo apt-get install -y imagemagick`

处理一张图片in.png，裁剪成300x280大小，从(30,0)作为裁剪的左上角点，得到out.png：

```
convert in.png -crop 300x280+30+0 out.png
```

其实这四个参数是我反复尝试二分法得到的，或许可以用专业软件快速得到吧

关键是可以批量处理呀，这里下载friends的头像图片进行处理：

```
for i in {1..79}; do curl -o $i.png http://kemono-friends.jp/wp-content/uploads/2016/11/no`printf "%03d" $i`.png --proxy socks5://127.0.0.1:1080; done
for i in {1..79}; do convert $i.png -crop 300x280+30+0 $i.png; done
```

其中使用了printf命令，可以使得1变成人家url需要的001

