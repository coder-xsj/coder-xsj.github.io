## 2019/07/14

Xshell远程连接

VMware快照功能



### bash shell

1. 命令

2. shell 脚本

   bash  ./scriptName

shell提示符

1.coder-xsj@coderxsj-1024:~$ 

​	用户名@主机名

​	～ 家目录

​	$ 普通账户

2.coderxsj-1024#

​	# root账户

切换用户: su - 用户名

创建用户: useradd test

设置密码: passwd test

~~~shell
coderxsj-1024# useradd test  
coderxsj-1024# passwd test
输入新的 UNIX 密码： 
重新输入新的 UNIX 密码： 
passwd：已成功更新密码
~~~

参数命令无法补全，装这个包

`apt-get install bash-completion `

#### bash快捷键

 ctrl+w 以空格为单位
ctrl+u 删除当前光标前内容
ctrl+k 删除当前光标后内容
ctrl+d 登出 exit或logout
ctrl+l 清屏 clear
ctrl+c 终止
ctrl+a 行首
ctrl+e 行尾
ctrl+z 挂起
ctrl+r 搜索历史命令



`#`注释

#### 历史记录

history	 查看当前shell所执行过的命令

!number 调用历史记录中第N个命令

查看历史记录中有关ping的命令

`history |grep ping`

`history -w `  #写到当前用户的家目录的 .bash_history 文件中

`esc + .  `获取上一命令的参数



#### 别名alias

`alias 别名=“命令”`

root@coderxsj-1024:~# alias pin=’ping www.baidu.com‘

释放别名 `unalias 别名`

root@coderxsj-1024:~# unalias pin

可以用`alias`查看当前系统的别名

~~~linux
coder-xsj@coderxsj-1024:~$ alias
alias alert='notify-send --urgency=low -i "$([ $? = 0 ] && echo terminal || echo error)" "$(history|tail -n1|sed -e '\''s/^\s*[0-9]\+\s*//;s/[;&|]\s*alert$//'\'')"'
alias egrep='egrep --color=auto'
alias fgrep='fgrep --color=auto'
alias grep='grep --color=auto'
alias l='ls -CF'
alias la='ls -A'
alias ll='ls -alF'
alias ls='ls --color=auto'
alias pin='ping www.baidu.com'

~~~

（ 临时）当前终端

##### 怎么使别名永久生效？

Ubuntu16.04中：vim /etc/bash.bashrc

修改用户目录下的一个文件 .bashrc 目录为 ~/.bashrc 

CentOs7: vim ~/.bashrc

alias tree='tree -C'

`source ~/.bashrc`

文本最后一行写入`alias pin='www.baidu.com'`



linux命令手册 <http://man.linuxde.net/>

另一个[http://linux.51yip.com](http://linux.51yip.com/)

内核：kernel.org



### 文件目录

etc：系统配置目录

网卡配置目录

~~~bash
root@coder-xsj ~]# vim /etc/sysconfig/network-scripts/ifcfg-
~~~

本地域名解析

~~~bash
[root@coder-xsj ~]# vim /etc/hosts
47.100.78.153 www.baidu.com
wq

[root@coder-xsj ~]# ping www.baidu.com
PING www.baidu.com (47.100.78.153) 56(84) bytes of data.
64 bytes from www.baidu.com (47.100.78.153): icmp_seq=1 ttl=64 time=0.183 ms
# 首先解析本地域名去了

~~~

系统主机名

~~~bash
coder-xsj@coderxsj-1024:~$ cat /etc/hostname 
coderxsj-1024
~~~

更改主机名

```bash
hostnamectl set-hostname xushengjin  # 使用这个命令会立即生效且重启也生效
```



var： 可变目录

/var/log:  系统日志文件

/var/tmp: 进程产生的 临时文件

ssh-keygen -t rsa -C “fibonacci701408733@163.com”

dev: 设备目录

/dev 存放设备文件，比如硬盘，硬盘分区，光盘（cdrom->sr0）

~~~bash
coder-xsj@coderxsj-1024:~$ ls /dev/|grep sd 
sda
sda1
sda2
sda3
sda5
sda6
sda7
sdb
sdb1
sdb2
sdb3
sdb4
sdb5
sdb6

~~~

/dev/null 黑洞设备，只进不出，类似于windows的回收站

/dev/random 生成随机数的设备

/dev/zero 摇钱树

产生一个G的数据

~~~bash
[root@coder-xsj ~]#  dd if=/dev/zero of=/opt/sys.iso bs=1M count=1024
记录了1024+0 的读入
记录了1024+0 的写出
1073741824字节(1.1 GB)已复制，8.5632 秒，125 MB/秒

~~~

proc：虚拟文件系统（进程停止则有的文件会被移除）

反应系统当前进程的实时状态

举个例子，查看httpd服务的pid号，然后停止它，再在/proc中寻找，看看还在不在

~~~bash
[root@coder-xsj ~]# pidof httpd
24500 24499 24498 24480 24479 24478 24477 24476 24474
# 在proc下确实有24500这个目录
[root@coder-xsj ~]# ls /proc/24500/
[root@coder-xsj ~]# systemctl stop httpd
[root@coder-xsj ~]# ls /proc/24500
ls: 无法访问/proc/24500: 没有那个文件或目录

~~~



注明：软链接形式

~~~
[root@coder-xsj ~]# ll /
lrwxrwxrwx.  1 root root     7 Jun 19 14:43 bin -> usr/bin
lrwxrwxrwx.  1 root root     7 Jun 19 14:43 lib -> usr/lib
lrwxrwxrwx.  1 root root     9 Jun 19 14:43 lib64 -> usr/lib64
lrwxrwxrwx.  1 root root     8 Jun 19 14:43 sbin -> usr/sbin
~~~



#### tree目录树

-C 颜色

-d 只显示目录

-L number  显示层级

~~~bash
#树状显示/下的二级目录带颜色
[root@coder-xsj ~]# tree -Ld 2 /
#树状显示home下的二级目录
coder-xsj@coderxsj-1024:~/centos$ tree -Ld 2 /home/
/home/
├── backup
├── coder-xsj
│   ├── anaconda3
│   ├── bao
│   ├── C-C++
│   ├── centos
│   ├── deepin-wine-for-ubuntu
│   ├── html
│   ├── jslx
........
38 directories


~~~



#### mkdir创建目录

-p：递归创建

-v：显示创建过程	

**{a1,b1}切记逗号之间不能加空格{a1, a2}**

```bash
#批量创建文件夹
coder-xsj@coderxsj-1024:~/centos$ mkdir -pv /tmp/test/{a1,b1}/{c1,d1}
mkdir: 已创建目录 '/tmp/test'
mkdir: 已创建目录 '/tmp/test/a1'
mkdir: 已创建目录 '/tmp/test/a1/c1'
mkdir: 已创建目录 '/tmp/test/a1/d1'
mkdir: 已创建目录 '/tmp/test/b1'
mkdir: 已创建目录 '/tmp/test/b1/c1'
mkdir: 已创建目录 '/tmp/test/b1/d1'
coder-xsj@coderxsj-1024:~/centos$ tree /tmp/test/
/tmp/test/
├── a1
│   ├── c1
│   └── d1
└── b1
    ├── c1
    └── d1

6 directories, 0 files

```



recursively: 递归

#### cp [选项]... 源文件... 目录

-i: 覆盖时交互

-v: 显示拷贝详情

-p: 保留源文件的属性

-r: 拷贝目录(递归方式)



##### 已知存在别名 alias cp='cp -i',怎么使用不带参数的cp命令呢?

1. /bin/cp
2. \cp

我若复制两次etc目录到tmp目录下,它就会提醒是否覆盖,而且是每一个文件提醒一次,我该怎么解决?

~~~bash
root@coderxsj-1024:~# cp -r /etc/ /tmp/
cp：是否覆盖'/tmp/etc/fstab'？ 
cp：是否覆盖'/tmp/etc/ImageMagick-6/coder.xml'？ 
cp：是否覆盖'/tmp/etc/ImageMagick-6/colors.xml'？ 
cp：是否覆盖'/tmp/etc/ImageMagick-6/delegates.xml'？ ^C
# 方法一,利用原本的命令
root@coderxsj-1024:~# /bin/cp -r /etc/ /tmp/
# 方法二,利用\符号,令命令失去其余参数
root@coderxsj-1024:~# \cp -r /etc/ /tmp/
~~~



#### Ubuntu16.04将小键盘锁自动开启

```
 sudo vim /usr/share/lightdm/lightdm.conf.d/50-unity-greeter.conf
```

在最后添加：greeter-setup-script=/usr/bin/numlockx on 
重启或者注销便可。启动时，便会发现小键盘灯自动打开。



mv 移动文件

移动目录不需要加什么-r参数,移就完事了

 **mv  源文件... 目录**

~~~bash
coderxsj-1024# mv Shazi.txt /opt 
# 若/opt存在,则相当于 
coderxsj-1024# mv Shazi.txt /opt/
# 若/opt不存在,则相当于重命名

~~~



cat查看文件

-n:显示行号

`<< >>`: 输入输出流

~~~bash
# 写数据
coder-xsj@coderxsj-1024:/var/www/html$ cat >> hello.txt << EOF
> hello world!
> EOF

~~~

注: EOF只是个标志,写什么都可以



head 查看

-n: 查看头几行,默认10行,-n可省略

~~~bash
coder-xsj@coderxsj-1024:/var/www/html$ head -n 5 /etc/services 
# Network services, Internet style
#
# Note that it is presently the policy of IANA to assign a single well-known
# port number for both TCP and UDP; hence, officially ports have two entries
# even if the protocol doesn't support UDP operations.

~~~

tail 查看尾行

-f: 实时查看

tail -f   == tailf



```sql
passthru,exec,system,chroot,chgrp,chown,shell_exec,popen,proc_open,pcntl_exec,ini_alter,ini_restore,dl,openlog,syslog,readlink,symlink,popepassthru,pcntl_alarm,pcntl_fork,pcntl_waitpid,pcntl_wait,pcntl_wifexited,pcntl_wifstopped,pcntl_wifsignaled,pcntl_wifcontinued,pcntl_wexitstatus,pcntl_wtermsig,pcntl_wstopsig,pcntl_signal,pcntl_signal_dispatch,pcntl_get_last_error,pcntl_strerror,pcntl_sigprocmask,pcntl_sigwaitinfo,pcntl_sigtimedwait,pcntl_exec,pcntl_getpriority,pcntl_setpriority,imap_open,apache_setenv
```

