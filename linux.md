### history

用于显示执行过的历史命令，能显示执行过的1000条命令。

### history -c

清空当前用户在本机上执行的 Linux 命令历史记录信息。

### pwd

显示用户当前所处的工作目录。

### whereis

whereis 命令用于按照名称快速搜索二进制程序（命令）、源代码以及帮助文件所对应的位置，语法格式为“whereis 命令名称”。

只能赋予普通用户具体的命令以满足工作需求，这也受到了必要的权限约束。如果需要让某个用户只能使用 root 管理员的身份执行指定的命令，切记一定要给出该命令的绝对路径，否则系统会识别不出来。这时，可以先使用 whereis 命令找出命令所对应的保存路径。

```bash
[root@linuxprobe~]# whereis cat

cat: /usr/bin/cat /usr/share/man/man1/cat.1.gz /usr/share/man/man1p/cat.1p.gz

[root@linuxprobe~]# whereis reboot

reboot: /usr/sbin/reboot /usr/share/man/man2/reboot.2.gz /usr/share/man/man8/reboot.8.gz
```

这里，cat的绝对路径为/usr/bin/cat，reboot的绝对路径为/usr/sbin/reboot。



### rm -r *
用于删除当前目录下的所有文件。

### rm -rf /*
删除Linux根目录包含的所有文件

### tar

**-c**

创建压缩文件

**-x**

解开压缩文件

**-z**

用gzip压缩或解压

**-v**

显示压缩或解压过程

**-f**

目标文件名**（必须放在参数的最后一位）**

**-p**

保留原始权限和属性

**-C**

指定解压到的目录**(只有解压需要使用)**

例如：先使用 tar 命令把/etc 目录通过 gzip格式进行打包压缩，并把文件命名为 etc.tar.gz。

#tar  czvf etc.tar.gz /etc

将打包后的压缩包文件指定解压到/root/etc 目录中（先使用 mkdir 命令创建/root/etc目录）

#mkdir /root/etc

#tar xzvf etc.tar.gz -C /root/etc

### 使用shell脚本时，可以传递参数

```bash
[root@linuxprobe~]# vim example.sh
#!/bin/bash
echo "当前脚本名称为$0"
echo "总共有$#个参数，分别是$*。"
echo "第 1 个参数为$1，第 5 个为$5。"
[root@linuxprobe~]# bash example.sh one two three four five six
当前脚本名称为 example.sh
总共有 6 个参数，分别是 one two three four five six。
第 1 个参数为 one，第 5 个为 five
```



[ 条件表达式 ] 若成立则返回数字0，否则返回非零值。

-d 

测试文件是否为目录类型

-e 

测试文件是否存在

-f 

判断是否为一般文件

-r 

测试当前用户是否有权限读取

-w 

测试当前用户是否有权限写入

-x 

测试当前用户是否有权限执行



**read 是用来读取用户输入信息的命令，能够把接收到的用户输入信息赋值给后面的指定变量，-p 参数用于向用户显示一些提示信息。**

### useradd - 创建并设置用户信息

创建指定的用户信息：

`[root@linuxcool ~]# useradd linuxprobe` 

创建指定的用户信息，但不创建家目录，亦不让登录系统（注： -M命令不建立用户家目录，-s命令设置用户的默认shell终端）： 

`[root@linuxcool ~]# useradd -M -s /sbin/nologin linuxprobe`


创建指定的用户信息，并自定义UID值：

`[root@linuxcool ~]# useradd -u 6688 linuxprobe`

创建指定的用户信息，并追加指定组为该用户的扩展组： 

`[root@linuxcool ~]# useradd -G root linuxprobe`

创建指定的用户信息，并指定过期时间： 

`[root@linuxcool ~]# useradd -e "2024/01/01" linuxprobe`



### setfacl 命令用于管理文件的 ACL 权限规则

-m 

修改权限

-M 

从文件中读取权限

-x 

删除某个权限

-b 

删除全部权限

-R 

递归子目录

例如，我们原本是无法进入/root 目录中的，现在为普通用户单独设置一下权限

`[root@linuxprobe~]# setfacl -Rm u:linuxprobe:rwx /root`

常用的 ls 命令是看不到 ACL 信息的，但是却可以看到文件权限的最后一个点(.)变成了加号(+)，这就意味着该文件已经设置了 ACL

```bash
[root@linuxprobe~]# ls -ld /root
dr-xrwx---+ 14 root root 4096 May 4 2020 /root
```



### iptables 中常用的参数以及作用

**-P** 

设置默认策略

**-F** 

清空规则链

**-L** 

查看规则链

**-A** 

在规则链的末尾加入新规则

**-I num** 

在规则链的头部加入新规则

**-D num** 

删除某一条规则

**-s** 

匹配来源地址 IP/MASK，加叹号“!”表示除这个 IP 外

**-d** 

匹配目标地址

**-i 网卡名称** 

匹配从这块网卡流入的数据

**-o 网卡名称** 

匹配从这块网卡流出的数据

**-p** 

匹配协议，如 TCP、UDP、ICMP

**-j**

ACCEPT、DROP、REJECT

**--dport num** 

匹配目标端口号

**--sport num** 

匹配来源端口号

例如：将input规则链设置为只允许指定网段的主机访问本机的22端口，拒绝来自其他网段所有主机的流量。

```bash
[root@linuxprobe~]# iptables -I INPUT -s 192.168.10.0/24 -p tcp --dport 22 -j ACCEPT

[root@linuxprobe~]# iptables -A INPUT -p tcp --dport 22 -j REJECT

[root@linuxprobe~]# iptables -L

Chain INPUT (policy ACCEPT)

target prot opt source destination

ACCEPT tcp -- 192.168.10.0/24 anywhere tcp dpt:ssh

REJECT tcp -- anywhere anywhere tcp dpt:ssh reject-with icmp-port-unreachable
```

防火墙策略规则是按照从上到下的顺序匹配的，因此一定要把允许动作放到拒绝动作前面，否则所有的流量就将被拒绝掉，从而导致任何主机都无法访问我们的服务。
