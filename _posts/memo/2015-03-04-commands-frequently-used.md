---
layout:     post
title:      "常用命令整理(更新必要)"
date:       2015-03-04 19:00:00 +0800
author:     "Jay Cai"
header-style: text
catalog: true
tags:
    - Shell
    - Hadoop
    - Spark
    - Kafka
    - K8S
---

## **Shell 命令**


### du
Linux `du`命令也是查看使用空间的，但是与`df`命令不同的是Linux `du`命令是对文件和目录磁盘使用的空间的查看，还是和`df`命令有一些区别的。

#### 命令格式
`du [选项][文件]`

#### 参数解释
*   -a或-all  显示目录中个别文件的大小。   
*   -b或-bytes  显示目录或文件大小时，以byte为单位。   
*   -c或--total  除了显示个别目录或文件的大小外，同时也显示所有目录或文件的总和。 
*   -k或--kilobytes  以KB(1024bytes)为单位输出。
*   -m或--megabytes  以MB为单位输出。   
*   -s或--summarize  仅显示总计，只列出最后加总的值。
*   -h或--human-readable  以K，M，G为单位，提高信息的可读性。
*   -x或--one-file-xystem  以一开始处理时的文件系统为准，若遇上其它不同的文件系统目录则略过。 
*   -L<符号链接>或--dereference<符号链接> 显示选项中所指定符号链接的源文件大小。   
*   -S或--separate-dirs   显示个别目录的大小时，并不含其子目录的大小。 
*   -X<文件>或--exclude-from=<文件>  在<文件>指定目录或文件。   
*   --exclude=<目录或文件>         略过指定的目录或文件。    
*   -D或--dereference-args   显示指定符号链接的源文件大小。   
*   -H或--si  与-h参数相同，但是K，M，G是以1000为换算单位。   
*   -l或--count-links   重复计算硬件链接的文件。  

#### 实例展示
```bash
du                        # 只显示当前目录下面的子目录的目录大小和当前目录的总的大小，最下面的1288为当前目录的总大小
du log2012.log            # 显示指定文件所占空间
du scf                    # 查看指定目录的所占空间
du log30.tar.gz log31.tar.gz          # 显示多个文件所占空间
du -s                                 # 只显示总和的大小
du -h test                            # 方便阅读的格式显示(大小以K,M,G来显示)
du -ah test                           # 文件和目录都显示
du -c log30.tar.gz log31.tar.gz       # -c 显示几个文件或目录各自占用磁盘空间的大小，还统计它们的总和
du|sort -nr|more                      # 按照空间大小排序
du -h  --max-depth=1                  # 输出当前目录下各个子目录所使用的空间
```

### df
linux中`df`命令的功能是用来检查linux服务器的文件系统的磁盘空间占用情况。可以利用该命令来获取硬盘被占用了多少空间，目前还剩下多少空间等信息。

#### 命令格式
`df [选项] [文件]`

#### 参数解释
必要参数：
*   -a 全部文件系统列表
*   -h 方便阅读方式显示
*   -H 等于“-h”，但是计算式，1K=1000，而不是1K=1024
*   -i 显示inode信息
*   -k 区块为1024字节
*   -l 只显示本地文件系统
*   -m 区块为1048576字节
*   --no-sync 忽略 sync 命令
*   -P 输出格式为POSIX
*   --sync 在取得磁盘信息前，先执行sync命令
*   -T 文件系统类型

可选择参数：
*   --block-size=<区块大小> 指定区块大小
*   -t<文件系统类型> 只显示选定文件系统的磁盘信息
*   -x<文件系统类型> 不显示选定文件系统的磁盘信息
*   --help 显示帮助信息
*   --version 显示版本信息

#### 实例展示
```bash
[root@CT1190 log]# df
文件系统               1K-块        已用     可用 已用% 挂载点
/dev/sda7             19840892    890896  17925856   5% /
/dev/sda9            203727156 112797500  80413912  59% /opt
/dev/sda8              4956284    570080   4130372  13% /var
/dev/sda6             19840892   1977568  16839184  11% /usr
/dev/sda3               988116     23880    913232   3% /boot
tmpfs                 16473212         0  16473212   0% /dev/shm
```
*   第1列是代表文件系统对应的设备文件的路径名（一般是硬盘上的分区）；
*   第2列给出分区包含的数据块（1024字节）的数目；
*   第3，4列分别表示已用的和可用的数据块数目。用户也许会感到奇怪的是，第3，4列块数之和不等于第2列中的块数。这是因为缺省的每个分区都留了少量空间供系统管理员使用。
    即使遇到普通用户空间已满的情况，管理员仍能登录和留有解决问题所需的工作空间。清单中Use% 列表示普通用户空间使用的百分比，即使这一数字达到100％，分区仍然留有系统管理员使用的空间。
*   最后，Mounted on列表示文件系统的挂载点。

```bash
df -i                     # 以inode模式来显示磁盘使用情况
df -t ext3                # 显示指定类型磁盘
df -ia                    # 列出各文件系统的i节点使用情况
df -T                     # 列出文件系统的类型
df -h                     # 以更易读的方式显示目前磁盘空间和使用情况
#   -h更具目前磁盘空间和使用情况 以更易读的方式显示
#   -H根上面的-h参数相同,不过在根式化的时候,采用1000而不是1024进行容量转换
#   -k以单位显示磁盘的使用情况
#   -l显示本地的分区的磁盘空间使用率,如果服务器nfs了远程服务器的磁盘,那么在df上加上-l后系统显示的是过滤nsf驱动器后的结果
#   -i显示inode的使用情况。linux采用了类似指针的方式管理磁盘空间影射.这也是一个比较关键应用
```

### ifconfig
`ifconfig`工具不仅可以被用来简单地获取网络接口配置信息，还可以修改这些配置。

#### 命令格式
`ifconfig [网络设备] [参数]`

#### 参数解释
*   up 启动指定网络设备/网卡。
*   down 关闭指定网络设备/网卡。该参数可以有效地阻止通过指定接口的IP信息流，如果想永久地关闭一个接口，我们还需要从核心路由表中将该接口的路由信息全部删除。
*   arp 设置指定网卡是否支持ARP协议。
*   -promisc 设置是否支持网卡的promiscuous模式，如果选择此参数，网卡将接收网络中发给它所有的数据包
*   -allmulti 设置是否支持多播模式，如果选择此参数，网卡将接收网络中所有的多播数据包
*   -a 显示全部接口信息
*   -s 显示摘要信息（类似于 netstat -i）
*   add 给指定网卡配置IPv6地址
*   del 删除指定网卡的IPv6地址
*   <硬件地址> 配置网卡最大的传输单元
*   mtu<字节数> 设置网卡的最大传输单元 (bytes)
*   netmask<子网掩码> 设置网卡的子网掩码。掩码可以是有前缀0x的32位十六进制数，也可以是用点分开的4个十进制数。如果不打算将网络分成子网，可以不管这一选项；如果要使用子网，那么请记住，网络中每一个系统必须有相同子网掩码。
*   tunel 建立隧道
*   dstaddr 设定一个远端地址，建立点对点通信
*   -broadcast<地址> 为指定网卡设置广播协议
*   -pointtopoint<地址> 为网卡设置点对点通讯协议
*   multicast 为网卡设置组播标志
*   address 为网卡设置IPv4地址
*   txqueuelen<长度> 为网卡设置传输列队的长度

#### 实例展示
```bash
[root@localhost ~]# ifconfig
eth0      Link encap:Ethernet  HWaddr 00:50:56:BF:26:20  
          inet addr:192.168.120.204  Bcast:192.168.120.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:8700857 errors:0 dropped:0 overruns:0 frame:0
          TX packets:31533 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:596390239 (568.7 MiB)  TX bytes:2886956 (2.7 MiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:68 errors:0 dropped:0 overruns:0 frame:0
          TX packets:68 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:2856 (2.7 KiB)  TX bytes:2856 (2.7 KiB)
```
*   eth0 表示第一块网卡， 其中 HWaddr 表示网卡的物理地址，可以看到目前这个网卡的物理地址(MAC地址）是 00:50:56:BF:26:20
*   inet addr 用来表示网卡的IP地址，此网卡的 IP地址是 192.168.120.204，广播地址， Bcast:192.168.120.255，掩码地址Mask:255.255.255.0 
*   lo 是表示主机的回坏地址，这个一般是用来测试一个网络程序，但又不想让局域网或外网的用户能够查看，只能在此台主机上运行和查看所用的网络接口。
    比如把 HTTPD服务器的指定到回坏地址，在浏览器输入 127.0.0.1就能看到你所架WEB网站了。但只是您能看得到，局域网的其它主机或用户无从知道。
*   第一行：连接类型：Ethernet（以太网）HWaddr（硬件mac地址）
*   第二行：网卡的IP地址、子网、掩码
*   第三行：UP（代表网卡开启状态）RUNNING（代表网卡的网线被接上）MULTICAST（支持组播）MTU:1500（最大传输单元）：1500字节
*   第四、五行：接收、发送数据包情况统计
*   第七行：接收、发送数据字节数统计信息。

```bash
# ssh登陆linux服务器操作要小心，关闭了就不能开启了，除非你有多网卡。
ifconfig eth0 up         # 启动网卡eth0
ifconfig eth0 down       # 关闭网卡eth0
ifconfig eth0 add 33ffe:3240:800:1005::2/64          # 为网卡eth0配置IPv6地址
ifconfig eth0 del 33ffe:3240:800:1005::2/64          # 为网卡eth0删除IPv6地址
ifconfig eth0 hw ether 00:AA:BB:CC:DD:EE             # 关闭网卡并修改MAC地址
ifconfig eth0 192.168.120.56 netmask 255.255.255.0 broadcast 192.168.120.255        # 给eth0网卡配置IP地址 子掩码 广播地址
ifconfig eth0 arp          # 开启网卡eth0 的arp协议
ifconfig eth0 -arp         # 关闭网卡eth0 的arp协议
ifconfig eth0 mtu 1480     # 设置能通过的最大数据包大小为 1500 bytes
```

### find
#### 命令格式
`find pathname -options [-print -exec -ok ...]`

#### 参数解释
*    `pathname`: find命令所查找的目录路径。例如用.来表示当前目录，用/来表示系统根目录。 
*    `-print`： find命令将匹配的文件输出到标准输出。 
*    `-exec`： find命令对匹配的文件执行该参数所给出的shell命令。相应命令的形式为`'command' {  } \;`，注意`{   }`和`\；`之间的空格。 
*    `-ok`： 和-exec的作用相同，只不过以一种更为安全的模式来执行该参数所给出的shell命令，在执行每一个命令之前，都会给出提示，让用户来确定是否执行。

#### 命令选项
*   -name   按照文件名查找文件。
*   -perm   按照文件权限来查找文件。
*   -prune  使用这一选项可以使find命令不在当前指定的目录中查找，如果同时使用-depth选项，那么-prune将被find命令忽略。
*   -user   按照文件属主来查找文件。
*   -group  按照文件所属的组来查找文件。
*   -mtime -n +n  按照文件的更改时间来查找文件， - n表示文件更改时间距现在n天以内，+ n表示文件更改时间距现在n天以前。find命令还有-atime和-ctime 选项，但它们都和-m time选项。
*   -nogroup  查找无有效所属组的文件，即该文件所属的组在/etc/groups中不存在。
*   -nouser   查找无有效属主的文件，即该文件的属主在/etc/passwd中不存在。
*   -newer file1 ! file2  查找更改时间比文件file1新但比文件file2旧的文件。
*   -type  查找某一类型的文件，诸如：
*   b - 块设备文件。
*   d - 目录。
*   c - 字符设备文件。
*   p - 管道文件。
*   l - 符号链接文件。
*   f - 普通文件。
*   -size n：[c] 查找文件长度为n块的文件，带有c时表示文件长度以字节计。-depth：在查找文件时，首先查找当前目录中的文件，然后再在其子目录中查找。
*   -fstype：查找位于某一类型文件系统中的文件，这些文件系统类型通常可以在配置文件/etc/fstab中找到，该配置文件中包含了本系统中有关文件系统的信息。
*   -mount：在查找文件时不跨越文件系统mount点。
*   -follow：如果find命令遇到符号链接文件，就跟踪至链接所指向的文件。
*   -cpio：对匹配的文件使用cpio命令，将这些文件备份到磁带设备中。
*   另外,下面三个的区别:
*   -amin n   查找系统中最后N分钟访问的文件
*   -atime n  查找系统中最后n\*24小时访问的文件
*   -cmin n   查找系统中最后N分钟被改变文件状态的文件
*   -ctime n  查找系统中最后n\*24小时被改变文件状态的文件
*   -mmin n   查找系统中最后N分钟被改变文件数据的文件
*   -mtime n  查找系统中最后n\*24小时被改变文件数据的文件

#### 基本实例
```bash
find -atime -2    # 超找48小时内修改过的文件
find . -name "*.log"         # 在当前目录查找 以.log结尾的文件。 ". "代表当前目录 
find /opt/soft/test/ -perm 777     # 查找/opt/soft/test/目录下 权限为 777的文件
find . -type f -name "*.log"       # 查找当目录，以.log结尾的普通文件 
find . -type d | sort              # 查找当前所有目录并排序
find . -size +1000c -print         # 在当前目录查找1000字节大小的文件
```
#### find之exec
*   -exec  参数后面跟的是command命令，它的终止是以;为结束标志的，所以这句命令后面的分号是不可缺少的，考虑到各个系统中分号会有不同的意义，所以前面加反斜杠。
*   {}   花括号代表前面find查找出来的文件名。
*   使用find时，只要把想要的操作写在一个文件里，就可以用`exec`来配合find查找，很方便的。在有些操作系统中只允许-exec选项执行诸如`ls`或`ls -l *`这样的命令。大多数用户使用这一选项是为了查找旧文件并删除它们。建议在真正执行`rm`命令删除文件之前，最好先用`ls`命令看一下，确认它们是所要删除的文件。 `exec`选项后面跟随着所要执行的命令或脚本，然后是一对儿`{ }`，一个空格和一个`\`，最后是一个分号。为了使用`exec`选项，必须要同时使用`print`选项。如果验证一下`find`命令，会发现该命令只输出从当前路径起的相对路径及文件名。

```bash
find . -type f -exec ls -l {} \;                # 匹配当前目录下的所有普通文件，并在-exec选项中使用ls -l命令将它们列出
find . -type f -mtime +14 -exec rm {} \;        # 在目录中查找更改时间在n日以前的文件并删除它们
find . -name "*.log" -mtime +5 -ok rm {} \;     # 在目录中查找更改时间在n日以前的文件并删除它们，在删除之前先给出提示
find /etc -name "passwd*" -exec grep "root" {} \;        # 首先匹配所有文件名为“ passwd*”的文件，例如passwd、passwd.old、passwd.bak，然后执行grep命令看看在这些文件中是否存在一个root用户
find . -name "*.log" -exec mv {} .. \;                   # 查找文件移动到指定目录(上一级目录)  
find . -name "*.log" -exec cp {} test \;                 # 查找文件拷贝到指定目录（test目录）  

```

#### find之xargs
在使用`find`命令的*-exec*选项处理匹配到的文件时，`find`命令将所有匹配到的文件一起传递给*exec*执行。但有些系统对能够传递给*exec*的命令长度有限制，这样在`find`命令运行几分钟之后，就会出现溢出错误。错误信息通常是“参数列太长”或“参数列溢出”。这就是*xargs*命令的用处所在，特别是与`find`命令一起使用。
`find`命令把匹配到的文件传递给*xargs*命令，而*xargs*命令每次只获取一部分文件而不是全部，不像*-exec*选项那样。这样它可以先处理最先获取的一部分文件，然后是下一批，并如此继续下去。
在有些系统中，使用*-exec*选项会为处理每一个匹配到的文件而发起一个相应的进程，并非将匹配到的文件全部作为参数一次执行；这样在有些情况下就会出现进程过多，系统性能下降的问题，因而效率不高； 而使用*xargs*命令则只有一个进程。另外，在使用*xargs*命令时，究竟是一次获取所有的参数，还是分批取得参数，以及每一次获取参数的数目都会根据该命令的选项及系统内核中相应的可调参数来确定。
```bash
find . -type f -print | xargs file                              # 查找系统中的每一个普通文件，然后使用xargs命令来测试它们分别属于哪类文件 
find / -name "core" -print | xargs echo "" >/tmp/core.log       # 在整个系统中查找内存信息转储文件(core dump) ，然后把结果保存到/tmp/core.log 文件中
find . -perm -7 -print | xargs chmod o-w                        # 在当前目录下查找所有用户具有读、写和执行权限的文件，并收回相应的写权限
find . -type f -print | xargs grep "hostname"                   # 用grep命令在所有的普通文件中搜索hostname这个词
find . -name \* -type f -print | xargs grep "hostnames"         # 用grep命令在当前目录下的所有普通文件中搜索hostnames这个词
find . -name "*.log" | xargs -i mv {} test                      # 使用xargs执行mv 
# find后执行xargs提示xargs: argument line too long解决方法：
find . -type f -atime +0 -print0 | xargs -0 -l1 -t rm -f        # -l1是一次处理一个；-t是处理之前打印出命令
find . -name "file" | xargs -I [] cp [] ..                      # 使用-i参数默认的前面输出用{}代替，-I参数可以指定其他代替字符，如例子中的[] 
find . -name "*.log" | xargs -p -i mv {} ..                     # -p参数会提示让你确认是否执行后面的命令,y执行，n不执行。
```

### free
`free`命令可以显示Linux系统中空闲的、已用的物理内存及*swap*内存,及被内核使用的*buffer*。在Linux系统监控的工具中，`free`命令是最经常使用的命令之一。

#### 命令格式
`free [参数]`

#### 参数解释
*   -b 　以Byte为单位显示内存使用情况。
*   -k 　以KB为单位显示内存使用情况。
*   -m 　以MB为单位显示内存使用情况。
*   -g   以GB为单位显示内存使用情况。
*   -o 　不显示缓冲区调节列。
*   -s<间隔秒数> 　持续观察内存使用状况。
*   -t 　显示内存总和列。
*   -V 　显示版本信息。

#### 实例展示
```bash
[root@SF1150 service]# free
             total       used       free     shared    buffers     cached
Mem:      32940112   30841684    2098428          0    4545340   11363424
-/+ buffers/cache:   14932920   18007192
Swap:     32764556    1944984   30819572
[root@SF1150 service]#
```
输出说明：
*   total:总计物理内存的大小。
*   used:已使用多大。
*   free:可用有多少。
*   Shared:多个进程共享的内存总额。
*   Buffers/cached:磁盘缓存的大小。
*   第三行(-/+ buffers/cached):
    *   used:已使用多大。
    *   free:可用有多少。
*   第四行是交换分区SWAP的，也就是我们通常所说的虚拟内存。

区别：第二行(*mem*)的*used/free*与第三行(*-/+ buffers/cache*) *used/free*的区别。 这两个的区别在于使用的角度来看，第二行是从OS的角度来看，因为对于OS，*buffers/cached*都是属于被使用，所以他的可用内存是*2098428KB*,已用内存是*30841684KB*,其中包括，内核（*OS*）使用+*Application(X, oracle,etc)*使用的+*buffers*+*cached*.
第三行所指的是从应用程序角度来看，对于应用程序来说，*buffers/cached*是等于可用的，因为*buffer/cached*是为了提高文件读取的性能，当应用程序需在用到内存的时候，*buffer/cached*会很快地被回收。
所以从应用程序的角度来说，可用内存=*系统free memory*+*buffers+cached*。
如本机情况的可用内存为：18007156=2098428KB+4545340KB+11363424KB

接下来解释什么时候内存会被交换，以及按什么方交换。
当可用内存少于额定值的时候，就会开会进行交换.如何看额定值：`cat /proc/meminfo`
交换将通过三个途径来减少系统中使用的物理页面的个数：
*  1\.减少缓冲与页面cache的大小
*  2\.将系统V类型的内存页面交换出去
*  3\.换出或者丢弃页面。(Application 占用的内存页，也就是物理内存不足）。

事实上，少量地使用swap是不会影响到系统性能的。
**那buffers和cached都是缓存，两者有什么区别呢？**
为了提高磁盘存取效率, Linux做了一些精心的设计, 除了对*dentry*进行缓存(用于*VFS*,加速文件路径名到inode的转换), 还采取了两种主要Cache方式：*Buffer Cache*和*Page Cache*。前者针对磁盘块的读写，后者针对文件inode的读写。这些Cache有效缩短了I/O系统调用(比如*read*,*write*,*getdents*)的时间。
磁盘的操作有逻辑级（文件系统）和物理级（磁盘块），这两种Cache就是分别缓存逻辑和物理级数据的。
*Page cache*实际上是针对文件系统的，是文件的缓存，在文件层面上的数据会缓存到*page cache*。文件的逻辑层需要映射到实际的物理磁盘，这种映射关系由文件系统来完成。当*page cache*的数据需要刷新时，*page cache*中的数据交给*buffer cache*，因为*Buffer Cache*就是缓存磁盘块的。但是这种处理在2.6版本的内核之后就变的很简单了，没有真正意义上的cache操作。
*Buffer cache*是针对磁盘块的缓存，也就是在没有文件系统的情况下，直接对磁盘进行操作的数据会缓存到*buffer cache*中，例如，文件系统的元数据都会缓存到*buffer cache*中。
简单说来，*page cache*用来缓存文件数据，*buffer cache*用来缓存磁盘数据。在有文件系统的情况下，对文件操作，那么数据会缓存到*page cache*，如果直接采用dd等工具对磁盘进行读写，那么数据会缓存到*buffer cache*。
所以我们看linux,只要不用swap的交换空间,就不用担心自己的内存太少.如果常常swap用很多,可能你就要考虑加物理内存了.这也是linux看内存是否够用的标准.
如果是应用服务器的话，一般只看第三行，*+buffers/cache*,即对应用程序来说*free*的内存太少了，也是该考虑优化程序或加内存了。
```bash
free -t          # 以总和的形式显示内存的使用信息
free -s 10       # 周期性(10s)的查询内存使用信息
```

### grep
Linux系统中`grep`命令是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹 配的行打印出来。`grep`全称是*Global Regular Expression Print*，表示全局正则表达式版本，它的使用权限是所有用户。
`grep`的工作方式是这样的，它在一个或多个文件中搜索字符串模板。如果模板包括空格，则必须被引用，模板后的所有字符串被看作文件名。搜索的结果被送到标准输出，不影响原文件内容。
`grep`可用于shell脚本，因为`grep`通过返回一个状态值来说明搜索的状态，如果模板搜索成功，则返回0，如果搜索不成功，则返回1，如果搜索的文件不存在，则返回2。我们利用这些返回值就可进行一些自动化的文本处理工作。

#### 命令格式
`grep [option] pattern file`

#### 参数解释
*   -a   --text   #不要忽略二进制的数据。   
*   -A<显示行数>   --after-context=<显示行数>   #除了显示符合范本样式的那一列之外，并显示该行之后的内容。   
*   -b   --byte-offset   #在显示符合样式的那一行之前，标示出该行第一个字符的编号。   
*   -B<显示行数>   --before-context=<显示行数>   #除了显示符合样式的那一行之外，并显示该行之前的内容。   
*   -c    --count   #计算符合样式的列数。   
*   -C<显示行数>    --context=<显示行数>或-<显示行数>   #除了显示符合样式的那一行之外，并显示该行之前后的内容。   
*   -d <动作>      --directories=<动作>   #当指定要查找的是目录而非文件时，必须使用这项参数，否则grep指令将回报信息并停止动作。   
*   -e<范本样式>  --regexp=<范本样式>   #指定字符串做为查找文件内容的样式。   
*   -E      --extended-regexp   #将样式为延伸的普通表示法来使用。   
*   -f<规则文件>  --file=<规则文件>   #指定规则文件，其内容含有一个或多个规则样式，让grep查找符合规则条件的文件内容，格式为每行一个规则样式。   
*   -F   --fixed-regexp   #将样式视为固定字符串的列表。   
*   -G   --basic-regexp   #将样式视为普通的表示法来使用。   
*   -h   --no-filename   #在显示符合样式的那一行之前，不标示该行所属的文件名称。   
*   -H   --with-filename   #在显示符合样式的那一行之前，表示该行所属的文件名称。   
*   -i    --ignore-case   #忽略字符大小写的差别。   
*   -l    --file-with-matches   #列出文件内容符合指定的样式的文件名称。   
*   -L   --files-without-match   #列出文件内容不符合指定的样式的文件名称。   
*   -n   --line-number   #在显示符合样式的那一行之前，标示出该行的列数编号。   
*   -q   --quiet或--silent   #不显示任何信息。   
*   -r   --recursive   #此参数的效果和指定“-d recurse”参数相同。   
*   -s   --no-messages   #不显示错误信息。   
*   -v   --revert-match   #显示不包含匹配文本的所有行。   
*   -V   --version   #显示版本信息。   
*   -w   --word-regexp   #只显示全字符合的列。   
*   -x    --line-regexp   #只显示全列符合的列。   
*   -y   #此参数的效果和指定“-i”参数相同。
*   grep正则表达式参考另外一个正则表达式的整理

#### 实例展示
```bash
ps -ef|grep svn                    # 查找指定进程
ps -ef|grep -c svn                 # 查找指定进程个数
cat test.txt | grep -f test2.txt   # 输出test.txt文件中含有从test2.txt文件中读取出的关键词的内容行
cat test.txt | grep -nf test2.txt  # 输出test.txt文件中含有从test2.txt文件中读取出的关键词的内容行，并显示每一行的行号
grep -n 'linux' test.txt           # 从文件中查找关键词 -n可以省略
grep 'linux' test.txt test2.txt    # 多文件时，输出查询到的信息内容行时，会把文件的命名在行最前面输出并且加上":"作为标示符
ps aux | grep ssh | grep -v "grep" # grep不显示本身进程
cat test.txt |grep ^u              # 找出已u开头的行内容
cat test.txt |grep ^[^u]           # 输出非u开头的行内容
cat test.txt |grep hat$            # 输出以hat结尾的行内容
cat test.txt |grep -E "ed|at"      # 显示包含ed或者at字符的内容行
grep '[a-z]\{7\}' *.txt            # 显示当前目录下面以.txt 结尾的文件中的所有包含每个字符串至少有7个连续小写字符的字符串的行
```

### ps
Linux中的`ps`命令是Process Status的缩写。`ps`命令用来列出系统中当前运行的那些进程。`ps`命令列出的是当前那些进程的快照，就是执行`ps`命令的那个时刻的那些进程，如果想要动态的显示进程信息，就可以使用`top`命令。
要对进程进行监测和控制，首先必须要了解当前进程的情况，也就是需要查看当前进程，而`ps`命令就是最基本同时也是非常强大的进程查看命令。使用该命令可以确定有哪些进程正在运行和运行的状态、进程是否结束、进程有没有僵死、哪些进程占用了过多的资源等等。总之大部分信息都是可以通过执行该命令得到的。
`ps`为我们提供了进程的一次性的查看，它所提供的查看结果并不动态连续的；如果想对进程时间监控，应该用`top`工具。
`kill`命令用于杀死进程。

**linux上进程有5种状态:**
1. 运行(正在运行或在运行队列中等待) 
2. 中断(休眠中, 受阻, 在等待某个条件的形成或接受到信号) 
3. 不可中断(收到信号不唤醒和不可运行, 进程必须等待直到有中断发生) 
4. 僵死(进程已终止, 但进程描述符存在, 直到父进程调用wait4()系统调用后释放) 
5. 停止(进程收到SIGSTOP, SIGSTP, SIGTIN, SIGTOU信号后停止运行运行) 

**ps工具标识进程的5种状态码:**
*   D 不可中断 uninterruptible sleep (usually IO) 
*   R 运行 runnable (on run queue) 
*   S 中断 sleeping 
*   T 停止 traced or stopped 
*   Z 僵死 a defunct (”zombie”) process 

#### 命令格式
`ps [参数]`

#### 参数解释
*   a  显示所有进程
*   -a 显示同一终端下的所有程序
*   -A 显示所有进程
*   c  显示进程的真实名称
*   -N 反向选择
*   -e 等于“-A”
*   e  显示环境变量
*   f  显示程序间的关系
*   -H 显示树状结构
*   r  显示当前终端的进程
*   T  显示当前终端的所有程序
*   u  指定用户的所有进程
*   -au 显示较详细的资讯
*   -aux 显示所有包含其他使用者的行程 
*   -C<命令> 列出指定命令的状况
*   --lines<行数> 每页显示的行数
*   --width<字符数> 每页显示的字符数
*   --help 显示帮助信息
*   --version 显示版本显示

#### 实例展示
```bash
ps -A           # 显示所有进程信息
ps -u root      # 显示指定用户信息
ps -ef          # 显示所有进程信息，连同命令行
ps -ef|grep ssh      # 查找特定进程
ps aux               # 列出目前所有的正在内存当中的程序
ps -axjf             # 列出类似程序树的程序显示
ps aux | egrep '(cron|syslog)' # 找出与 cron 与 syslog 这两个服务有关的 PID 号码
ps -aux |more                  # 用 | 管道和 more 连接起来分页查看
ps -aux > ps001.txt            # 把所有进程显示出来，并输出到ps001.txt文件
ps -o pid,ppid,pgrp,session,tpgid,comm    # 输出指定的字段
```
```bash
 # 将目前属于您自己这次登入的 PID 与相关信息列示出来
[root@localhost test6]# ps -l
F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
4 S     0 17398 17394  0  75   0 - 16543 wait   pts/0    00:00:00 bash
4 R     0 17469 17398  0  77   0 - 15877 -      pts/0    00:00:00 ps
# 各相关信息的意义：
#    F 代表这个程序的旗标 (flag)， 4 代表使用者为 super user
#    S 代表这个程序的状态 (STAT)，关于各 STAT 的意义将在内文介绍
#    UID 程序被该 UID 所拥有
#    PID 就是这个程序的 ID ！
#    PPID 则是其上级父程序的ID
#    C CPU 使用的资源百分比
#    PRI 这个是 Priority (优先执行序) 的缩写，详细后面介绍
#    NI 这个是 Nice 值，在下一小节我们会持续介绍
#    ADDR 这个是 kernel function，指出该程序在内存的那个部分。如果是个 running的程序，一般就是 "-"
#    SZ 使用掉的内存大小
#    WCHAN 目前这个程序是否正在运作当中，若为 - 表示正在运作
#    TTY 登入者的终端机位置
#    TIME 使用掉的 CPU 时间。
#    CMD 所下达的指令为何
#    在预设的情况下， ps 仅会列出与目前所在的 bash shell 有关的 PID 而已，所以， 当我使用 ps -l 的时候，只有三个 PID。
```
```bash
#  列出目前所有的正在内存当中的程序
[root@localhost test6]# ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0  10368   676 ?        Ss   Nov02   0:00 init [3]                  
root         2  0.0  0.0      0     0 ?        S<   Nov02   0:01 [migration/0]
root         3  0.0  0.0      0     0 ?        SN   Nov02   0:00 [ksoftirqd/0]
root         4  0.0  0.0      0     0 ?        S<   Nov02   0:01 [migration/1]
··· ···
··· ···
# 各相关信息的意义：
#   USER：该 process 属于那个使用者账号的
#   PID ：该 process 的号码
#   %CPU：该 process 使用掉的 CPU 资源百分比
#   %MEM：该 process 所占用的物理内存百分比
#   VSZ ：该 process 使用掉的虚拟内存量 (Kbytes)
#   RSS ：该 process 占用的固定的内存量 (Kbytes)
#   TTY ：该 process 是在那个终端机上面运作，若与终端机无关，则显示 ?，另外， tty1-tty6 是本机上面的登入者程序，若为 pts/0 等等的，则表示为由网络连接进主机的程序。
#   STAT：该程序目前的状态，主要的状态有
#   R ：该程序目前正在运作，或者是可被运作
#   S ：该程序目前正在睡眠当中 (可说是 idle 状态)，但可被某些讯号 (signal) 唤醒。
#   T ：该程序目前正在侦测或者是停止了
#   Z ：该程序应该已经终止，但是其父程序却无法正常的终止他，造成 zombie (疆尸) 程序的状态
#   START：该 process 被触发启动的时间
#   TIME ：该 process 实际使用 CPU 运作的时间
#   COMMAND：该程序的实际指令
```

### netstat
`netstat`命令用于显示与*IP*、*TCP*、*UDP*和*ICMP*协议相关的统计数据，一般用于检验本机各端口的网络连接情况。`netstat`是在内核中访问网络及相关信息的程序，它能提供*TCP*连接，*TCP*和*UDP*监听，进程内存管理的相关报告。
如果你的计算机有时候接收到的数据报导致出错数据或故障，你不必感到奇怪，*TCP/IP*可以容许这些类型的错误，并能够自动重发数据报。但如果累计的出错情况数目占到所接收的IP数据报相当大的百分比，或者它的数目正迅速增加，那么你就应该使用`netstat`查一查为什么会出现这些情况了

#### 命令格式
`netstat [-acCeFghilMnNoprstuvVwx][-A<网络类型>][--ip]`

#### 参数解释
*   -a或–all 显示所有连线中的Socket。
*   -A<网络类型>或–<网络类型> 列出该网络类型连线中的相关地址。
*   -c或–continuous 持续列出网络状态。
*   -C或–cache 显示路由器配置的快取信息。
*   -e或–extend 显示网络其他相关信息。
*   -F或–fib 显示FIB。
*   -g或–groups 显示多重广播功能群组组员名单。
*   -h或–help 在线帮助。
*   -i或–interfaces 显示网络界面信息表单。
*   -l或–listening 显示监控中的服务器的Socket。
*   -M或–masquerade 显示伪装的网络连线。
*   -n或–numeric 直接使用IP地址，而不通过域名服务器。
*   -N或–netlink或–symbolic 显示网络硬件外围设备的符号连接名称。
*   -o或–timers 显示计时器。
*   -p或–programs 显示正在使用Socket的程序识别码和程序名称。
*   -r或–route 显示Routing Table。
*   -s或–statistice 显示网络工作信息统计表。
*   -t或–tcp 显示TCP传输协议的连线状况。
*   -u或–udp 显示UDP传输协议的连线状况。
*   -v或–verbose 显示指令执行过程。
*   -V或–version 显示版本信息。
*   -w或–raw 显示RAW传输协议的连线状况。
*   -x或–unix 此参数的效果和指定”-A unix”参数相同。
*   –ip或–inet 此参数的效果和指定”-A inet”参数相同。

#### 实例展示
```bash
[root@localhost ~]# netstat
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State      
tcp        0    268 192.168.120.204:ssh         10.2.0.68:62420             ESTABLISHED 
udp        0      0 192.168.120.204:4371        10.58.119.119:domain        ESTABLISHED 
Active UNIX domain sockets (w/o servers)
Proto RefCnt Flags       Type       State         I-Node Path
unix  2      [ ]         DGRAM                    1491   @/org/kernel/udev/udevd
unix  4      [ ]         DGRAM                    7337   /dev/log
unix  2      [ ]         DGRAM                    708823 
unix  2      [ ]         DGRAM                    7539   
unix  3      [ ]         STREAM     CONNECTED     7287   
unix  3      [ ]         STREAM     CONNECTED     7286   
[root@localhost ~]#
```
说明：
从整体上看，netstat的输出结果可以分为两个部分：
一个是Active Internet connections，称为有源TCP连接，其中"Recv-Q"和"Send-Q"指的是接收队列和发送队列。这些数字一般都应该是0。如果不是则表示软件包正在队列中堆积。这种情况只能在非常少的情况见到。
另一个是Active UNIX domain sockets，称为有源Unix域套接口(和网络套接字一样，但是只能用于本机通信，性能可以提高一倍)。
Proto显示连接使用的协议,RefCnt表示连接到本套接口上的进程号,Types显示套接口的类型,State显示套接口当前的状态,Path表示连接到套接口的其它进程使用的路径名。

套接口类型：
-t ：TCP
-u ：UDP
-raw ：RAW类型
--unix ：UNIX域类型
--ax25 ：AX25类型
--ipx ：ipx类型
--netrom ：netrom类型
状态说明：
LISTEN：侦听来自远方的TCP端口的连接请求
SYN-SENT：再发送连接请求后等待匹配的连接请求（如果有大量这样的状态包，检查是否中招了）
SYN-RECEIVED：再收到和发送一个连接请求后等待对方对连接请求的确认（如有大量此状态，估计被flood攻击了）
ESTABLISHED：代表一个打开的连接
FIN-WAIT-1：等待远程TCP连接中断请求，或先前的连接中断请求的确认
FIN-WAIT-2：从远程TCP等待连接中断请求
CLOSE-WAIT：等待从本地用户发来的连接中断请求
CLOSING：等待远程TCP对连接中断的确认
LAST-ACK：等待原来的发向远程TCP的连接中断请求的确认（不是什么好东西，此项出现，检查是否被攻击）
TIME-WAIT：等待足够的时间以确保远程TCP接收到连接中断请求的确认
CLOSED：没有任何连接状态

```bash
netstat -a          # 显示一个所有的有效连接信息列表，包括已建立的连接（ESTABLISHED），也包括监听连接请（LISTENING）的那些连接。
netstat -nu         # 显示当前UDP连接状况
netstat -apu        # 显示UDP端口号的使用情况
netstat -i          # 显示网卡列表
netstat -g          # 显示组播组的关系
netstat -s          # 显示网络统计信息
netstat -l          # 显示监听的套接口
netstat -n          # 显示所有已建立的有效连接
netstat -e          # 显示关于以太网的统计数据.它列出的项目包括传送的数据报的总字节数、错误数、删除数、数据报的数量和广播的数量。这些统计数据既有发送的数据报数量，也有接收的数据报数量。这个选项可以用来统计一些基本的网络流量）
netstat -r          # 显示关于路由表的信息
netstat -at         # 列出所有 tcp 端口
netstat -a | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'      # 统计机器中网络连接各个状态个数
netstat -nat |awk '{print $6}'|sort|uniq -c                               # 把状态全都取出来后使用uniq -c统计后再进行排序
netstat -nat | grep "192.168.120.20:16067" |awk '{print $5}'|awk -F: '{print $4}'|sort|uniq -c|sort -nr|head -20      # 查看连接某服务端口最多的的IP地址
netstat -ap | grep ssh     # 找出程序运行的端口
netstat -pt                # 在 netstat 输出中显示 PID 和进程名称
netstat -anpt | grep ':16064'  # netstat -anpt | grep ':16064'
```

### scp
`scp`是secure copy的简写，用于在Linux下进行远程拷贝文件的命令，和它类似的命令有`cp`，不过`cp`只是在本机进行拷贝不能跨服务器，而且scp传输是加密的。可能会稍微影响一下速度。当你服务器硬盘变为只读*read only system*时，用`scp`可以帮你把文件移出来。另外，`scp`还非常不占资源，不会提高多少系统负荷，在这一点上，`rsync`就远远不及它了。虽然`rsync`比scp会快一点，但当小文件众多的情况下，`rsync`会导致硬盘I/O非常高，而`scp`基本不影响系统正常使用。
#### 命令格式
`scp [参数] [原路径] [目标路径]`

#### 参数解释
*  -1  强制scp命令使用协议ssh1  
*  -2  强制scp命令使用协议ssh2  
*  -4  强制scp命令只使用IPv4寻址  
*  -6  强制scp命令只使用IPv6寻址  
*  -B  使用批处理模式（传输过程中不询问传输口令或短语）  
*  -C  允许压缩。（将-C标志传递给ssh，从而打开压缩功能）  
*  -p 保留原文件的修改时间，访问时间和访问权限。  
*  -q  不显示传输进度条。  
*  -r  递归复制整个目录。  
*  -v 详细方式显示输出。scp和ssh(1)会显示出整个过程的调试信息。这些信息用于调试连接，验证和配置问题。   
*  -c cipher  以cipher将数据传输进行加密，这个选项将直接传递给ssh。   
*  -F ssh_config  指定一个替代的ssh配置文件，此参数直接传递给ssh。  
*  -i identity_file  从指定文件中读取传输时使用的密钥文件，此参数直接传递给ssh。    
*  -l limit  限定用户所能使用的带宽，以Kbit/s为单位。     
*  -o ssh_option  如果习惯于使用ssh_config(5)中的参数传递方式，   
*  -P port  注意是大写的P, port是指定数据传输用到的端口号   
*  -S program  指定加密传输时所使用的程序。此程序必须能够理解ssh(1)的选项。

#### 实例展示
```bash
# 复制本地文件到远程服务器
scp local_file remote_username@remote_ip:remote_folder
scp local_file remote_username@remote_ip:remote_file
scp local_file remote_ip:remote_folder
scp local_file remote_ip:remote_file

# 复制本地目录到远程服务器
scp -r local_folder remote_username@remote_ip:remote_folder
scp -r local_folder remote_ip:remote_folder 
```

### ss
`ss`是Socket Statistics的缩写。顾名思义，`ss`命令可以用来获取*socket*统计信息，它可以显示和`netstat`类似的内容。但`ss`的优势在于它能够显示更多更详细的有关*TCP*和连接状态的信息，而且比`netstat`更快速更高效。
当服务器的socket连接数量变得非常大时，无论是使用`netstat`命令还是直接`cat /proc/net/tcp`，执行速度都会很慢。可能你不会有切身的感受，但请相信我，当服务器维持的连接达到上万个的时候，使用`netstat`等于浪费 生命，而用`ss`才是节省时间。
天下武功唯快不破。`ss`快的秘诀在于，它利用到了*TCP*协议栈中*tcp_diag*。*tcp_diag*是一个用于分析统计的模块，可以获得Linux 内核中第一手的信息，这就确保了`ss`的快捷高效。当然，如果你的系统中没有*tcp_diag*，`ss`也可以正常运行，只是效率会变得稍慢。（但仍然比`netstat`要快。）

#### 命令格式
`ss [参数]`
`ss [参数] [过滤]`

#### 参数解释
*   -h, --help  帮助信息
*   -V, --version   程序版本信息
*   -n, --numeric   不解析服务名称
*   -r, --resolve        解析主机名
*   -a, --all   显示所有套接字（sockets）
*   -l, --listening 显示监听状态的套接字（sockets）
*   -o, --options        显示计时器信息
*   -e, --extended       显示详细的套接字（sockets）信息
*   -m, --memory         显示套接字（socket）的内存使用情况
*   -p, --processes 显示使用套接字（socket）的进程
*   -i, --info  显示 TCP内部信息
*   -s, --summary   显示套接字（socket）使用概况
*   -4, --ipv4           仅显示IPv4的套接字（sockets）
*   -6, --ipv6           仅显示IPv6的套接字（sockets）
*   -0, --packet            显示 PACKET 套接字（socket）
*   -t, --tcp   仅显示 TCP套接字（sockets）
*   -u, --udp   仅显示 UCP套接字（sockets）
*   -d, --dccp  仅显示 DCCP套接字（sockets）
*   -w, --raw   仅显示 RAW套接字（sockets）
*   -x, --unix  仅显示 Unix套接字（sockets）
*   -f, --family=FAMILY  显示 FAMILY类型的套接字（sockets），FAMILY可选，支持  unix, inet, inet6, link, netlink
*   -A, --query=QUERY, --socket=QUERY
          QUERY := {all|inet|tcp|udp|raw|unix|packet|netlink}\[,QUERY\]
*   -D, --diag=FILE     将原始TCP套接字（sockets）信息转储到文件
*   -F, --filter=FILE   从文件中都去过滤器信息
      FILTER := \[ state TCP-STATE \] \[ EXPRESSION \]

#### 实例展示
```bash
ss -t -a                # 显示TCP连接
ss -s                   # 显示 Sockets 摘要
ss -l                   # 列出所有打开的网络连接端
ss -pl                  # 查看进程使用的socket
ss -lp | grep 3306      # 找出打开套接字/端口应用程序
ss -u -a                # 显示所有UDP Sockets
ss -o state established '( dport = :smtp or sport = :smtp )'      # 显示所有状态为established的SMTP连接
ss -o state established '( dport = :http or sport = :http )'      #显示所有状态为Established的HTTP连接
ss -o state fin-wait-1 '( sport = :http or sport = :https )' dst 193.233.7/24     # 列举出处于 FIN-WAIT-1状态的源端口为 80或者 443，目标网络为 193.233.7/24所有 tcp套接字
ss -4 state closing      # 用TCP 状态过滤Sockets:
# 匹配远程地址和端口号
ss dst ADDRESS_PATTERN
ss dst 192.168.119.113
ss dst 192.168.119.113:http
ss dst 192.168.119.113:smtp
ss dst 192.168.119.113:3844
# 匹配本地地址和端口号
ss src ADDRESS_PATTERN
ss src 192.168.119.103:16021
```

### tar
#### 命令格式
`tar[必要参数][选择参数][文件]`

#### 参数解释
必要参数有如下：

*   -A 新增压缩文件到已存在的压缩
*   -B 设置区块大小
*   -c 建立新的压缩文件
*   -d 记录文件的差别
*   -r 添加文件到已经压缩的文件
*   -u 添加改变了和现有的文件到已经存在的压缩文件
*   -x 从压缩的文件中提取文件
*   -t 显示压缩文件的内容
*   -z 支持gzip解压文件
*   -j 支持bzip2解压文件
*   -Z 支持compress解压文件
*   -v 显示操作过程
*   -l 文件系统边界设置
*   -k 保留原有文件不覆盖
*   -m 保留文件不被覆盖
*   -W 确认压缩文件的正确性

可选参数如下：
*   -b 设置区块数目
*   -C 切换到指定目录
*   -f 指定压缩文件
*   --help 显示帮助信息
*   --version 显示版本信息

#### 实例展示
```bash
tar xvf FileName.tar                # 解包
tar cvf FileName.tar DirName        # 打包
gunzip FileName.gz                  # 解压
# .gz
gzip -d FileName.gz                 # 解压
gzip FileName                       # 压缩
#.tar.gz
tar zxvf FileName.tar.gz            # 解压
tar zcvf FileName.tar.gz DirName    # 压缩
# .bz2
bzip2 -d FileName.bz2               # 解压
bunzip2 FileName.bz2                # 解压
bzip2 -z FileName                   # 压缩
#   .tar.bz2
tar jxvf FileName.tar.bz2           # 解压
tar jcvf FileName.tar.bz2 DirName   # 压缩
#   .bz
bzip2 -d FileName.bz                # 解压
bunzip2 FileName.bz                 # 解压
#   .tar.bz
tar jxvf FileName.tar.bz            # 解压
#   .Z
uncompress FileName.Z                 # 解压
compress FileName                     # 压缩
#   .tar.Z
tar Zxvf FileName.tar.Z               # 解压
tar Zcvf FileName.tar.Z DirName       # 压缩
#   .zip
unzip FileName.zip                    # 解压
zip FileName.zip DirName              # 压缩
#   .rar
rar x FileName.rar                    # 解压
rar a FileName.rar DirName            # 压缩


tar -cvf log.tar log2012.log         # 仅打包，不压缩！
tar -zcvf log.tar.gz log2012.log     # 打包后，以 gzip 压缩

tar -ztvf log.tar.gz                 # 查阅log.tar.gz包内的文件 只查看不解压
tar -zxvf /opt/soft/test/log.tar.gz  # 将tar 包解压缩

tar -zxvf /opt/soft/test/log30.tar.gz log2013.log    # 只将 tar 内的部分文件解压出来

tar -zcvpf log31.tar.gz log2014.log log2015.log log2016.log    # 文件备份下来，并且保存其权限`-p`
tar -N "2012/11/13" -zcvf log17.tar.gz test                    # 在test文件夹当中，比某个日期新的文件才备份
tar --exclude scf/service -zcvf scf.tar.gz scf/*               # 备份文件夹内容时排除部分文件
```

### top
#### 命令格式
`top [参数]`

#### 参数解释
*   -b 批处理
*   -c 显示完整的治命令
*   -I 忽略失效过程
*   -s 保密模式
*   -S 累积模式
*   -i<时间> 设置间隔时间
*   -u<用户名> 指定用户名
*   -p<进程号> 指定进程
*   -n<次数> 循环显示的次数

#### 实例展示
```bash
[root@TG1704 log]# top
top - 14:06:23 up 70 days, 16:44,  2 users,  load average: 1.25, 1.32, 1.35
Tasks: 206 total,   1 running, 205 sleeping,   0 stopped,   0 zombie
Cpu(s):  5.9%us,  3.4%sy,  0.0%ni, 90.4%id,  0.0%wa,  0.0%hi,  0.2%si,  0.0%st
Mem:  32949016k total, 14411180k used, 18537836k free,   169884k buffers
Swap: 32764556k total,        0k used, 32764556k free,  3612636k cached

  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND                                                                
28894 root      22   0 1501m 405m  10m S 52.2  1.3   2534:16 java                                                                   
18249 root      18   0 3201m 1.9g  11m S 35.9  6.0 569:39.41 java                                                                   
29062 root      20   0 1241m 227m  10m S  0.3  0.7   2:07.32 java                                                                   
    1 root      15   0 10368  684  572 S  0.0  0.0   1:30.85 init                                                                   
    2 root      RT  -5     0    0    0 S  0.0  0.0   0:01.01 migration/0                                                            
    3 root      34  19     0    0    0 S  0.0  0.0   0:00.00 ksoftirqd/0                                                            
    4 root      RT  -5     0    0    0 S  0.0  0.0   0:00.00 watchdog/0                                                             
    5 root      RT  -5     0    0    0 S  0.0  0.0   0:00.80 migration/1                                                            
    6 root      34  19     0    0    0 S  0.0  0.0   0:00.00 ksoftirqd/1                                                            
```
统计信息区：
前五行是当前系统情况整体的统计信息区。下面我们看每一行信息的具体意义。
*   第一行，任务队列信息，同 uptime 命令的执行结果，具体参数说明情况如下：
    *   14:06:23 — 当前系统时间
    *   up 70 days, 16:44 — 系统已经运行了70天16小时44分钟（在这期间系统没有重启过的吆！）
    *   2 users — 当前有2个用户登录系统
    *   load average: 1.15, 1.42, 1.44 — load average后面的三个数分别是1分钟、5分钟、15分钟的负载情况。
    *   load average数据是每隔5秒钟检查一次活跃的进程数，然后按特定算法计算出的数值。如果这个数除以逻辑CPU的数量，结果高于5的时候就表明系统在超负荷运转了。
*   第二行，Tasks — 任务（进程），具体信息说明如下：
        系统现在共有206个进程，其中处于运行中的有1个，205个在休眠（sleep），stoped状态的有0个，zombie状态（僵尸）的有0个。
*   第三行，cpu状态信息，具体属性说明如下：
    *   5.9%us — 用户空间占用CPU的百分比。
    *   3.4% sy — 内核空间占用CPU的百分比。
    *   0.0% ni — 改变过优先级的进程占用CPU的百分比
    *   90.4% id — 空闲CPU百分比
    *   0.0% wa — IO等待占用CPU的百分比
    *   0.0% hi — 硬中断（Hardware IRQ）占用CPU的百分比
    *   0.2% si — 软中断（Software Interrupts）占用CPU的百分比
    *   备注：在这里CPU的使用比率和windows概念不同，需要理解linux系统用户空间和内核空间的相关知识！
*   第四行,内存状态，具体信息如下：
    *   32949016k total — 物理内存总量（32GB）
    *   14411180k used — 使用中的内存总量（14GB）
    *   18537836k free — 空闲内存总量（18GB）
    *   169884k buffers — 缓存的内存量 （169M）
*   第五行，swap交换分区信息，具体信息说明如下：
    *   32764556k total — 交换区总量（32GB）
    *   0k used — 使用的交换区总量（0K）
    *   32764556k free — 空闲交换区总量（32GB）
    *   3612636k cached — 缓冲的交换区总量（3.6GB）
    *   备注：
        第四行中使用中的内存总量（used）指的是现在系统内核控制的内存数，空闲内存总量（free）是内核还未纳入其管控范围的数量。纳入内核管理的内存不见得都在使用中，还包括过去使用过的现在可以被重复利用的内存，内核并不把这些可被重新使用的内存交还到free中去，因此在linux上free内存会越来越少，但不用为此担心。
        如果出于习惯去计算可用内存数，这里有个近似的计算公式：第四行的free + 第四行的buffers + 第五行的cached，按这个公式此台服务器的可用内存：18537836k +169884k +3612636k = 22GB左右。
        对于内存监控，在top里我们要时刻监控第五行swap交换分区的used，如果这个数值在不断的变化，说明内核在不断进行内存和swap的数据交换，这是真正的内存不够用了。
*   第六行，空行。
*   第七行以下：各进程（任务）的状态监控，项目列信息说明如下：
    *   PID — 进程id
    *   USER — 进程所有者
    *   PR — 进程优先级
    *   NI — nice值。负值表示高优先级，正值表示低优先级
    *   VIRT — 进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES
    *   RES — 进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
    *   SHR — 共享内存大小，单位kb
    *   S — 进程状态。D=不可中断的睡眠状态 R=运行 S=睡眠 T=跟踪/停止 Z=僵尸进程
    *   %CPU — 上次更新到现在的CPU时间占用百分比
    *   %MEM — 进程使用的物理内存百分比
    *   TIME+ — 进程使用的CPU时间总计，单位1/100秒
    *   COMMAND — 进程名称（命令名/命令行）

#### 使用技巧
1.  多U多核CPU监控
    在top基本视图中，按键盘数字“1”，可监控每个逻辑CPU的状况：
    ![](/img/posts/linux/top1.jpg)
    观察上图，服务器有16个逻辑CPU，实际上是4个物理CPU。再按数字键1，就会返回到top基本视图界面。

2.  高亮显示当前运行进程
    敲击键盘“b”（打开/关闭加亮效果），top的视图变化如下：
    ![](/img/posts/linux/top2.png)
    我们发现进程id为2570的“top”进程被加亮了，top进程就是视图第二行显示的唯一的运行态（runing）的那个进程，可以通过敲击“y”键关闭或打开运行态进程的加亮效果。

3.  进程字段排序
    默认进入top时，各进程是按照CPU的占用量来排序的，在下图中进程ID为28894的java进程排在第一（cpu占用142%），进程ID为574的java进程排在第二（cpu占用16%）。
    ![](/img/posts/linux/top3.png)
    敲击键盘“x”（打开/关闭排序列的加亮效果），top的视图变化如下：
    ![](/img/posts/linux/top4.png)
    可以看到，top默认的排序列是“%CPU”。

4.  通过”shift + >”或”shift + <”可以向右或左改变排序列
    下图是按一次”shift + >”的效果图,视图现在已经按照%MEM来排序。
    ![](/img/posts/linux/top5.png)

```bash
top -c             # 显示完整命令
top -b             # 以批处理模式显示程序信息
top -S             # 以累积模式显示程序信息
top -n 2           # 表示更新两次后终止更新显示
top -d 3           # 表示更新周期为3秒
top -p 574         # 显示指定的进程信息
```

#### top交互命令
在top 命令执行过程中可以使用的一些交互命令。这些命令都是单字母的，如果在命令行中使用了*s*选项， 其中一些命令可能会被屏蔽。
*   h 显示帮助画面，给出一些简短的命令总结说明
*   k 终止一个进程。
*   i 忽略闲置和僵死进程。这是一个开关式命令。
*   q 退出程序
*   r 重新安排一个进程的优先级别
*   S 切换到累计模式
*   s 改变两次刷新之间的延迟时间（单位为*s*），如果有小数，就换算成*ms*。输入0值则系统将不断刷新，默认值是*5s*
*   f或者F 从当前显示中添加或者删除项目
*   o或者O 改变显示项目的顺序
*   l 切换显示平均负载和启动时间信息
*   m 切换显示内存信息
*   t 切换显示进程和CPU状态信息
*   c 切换显示命令名称和完整命令行
*   M 根据驻留内存大小进行排序
*   P 根据CPU使用百分比大小进行排序
*   T 根据时间/累计时间进行排序
*   W 将当前设置写入`~/.toprc`文件中 


### watch
watch是一个非常实用的命令，基本所有的Linux发行版都带有这个小工具，如同名字一样，watch可以帮你监测一个命令的运行结果，省得你一遍遍的手动运行。在Linux下，watch是周期性的执行下个程序，并全屏显示执行结果。你可以拿他来监测你想要的一切命令的结果变化，比如 tail 一个 log 文件，ls 监测某个文件的大小变化，看你的想象力了！
#### 命令格式
`watch[参数][命令]`

#### 参数解释
必要参数有如下：

*   -n或--interval  watch缺省每2秒运行一下程序，可以用-n或-interval来指定间隔的时间。
*   -d或--differences  用-d或--differences 选项watch 会高亮显示变化的区域。 而-d=cumulative选项会把变动过的地方(不管最近的那次有没有变动)都高亮显示出来。
*   -t 或-no-title  会关闭watch命令在顶部的时间间隔,命令，当前时间的输出。
*   -h, --help 查看帮助文档

#### 实例展示
```bash
watch -n 1 -d netstat -ant         # 每隔一秒高亮显示网络链接数的变化情况
watch -n 1 -d 'pstree|grep http'   # 每隔一秒高亮显示http链接数的变化情况
watch 'netstat -an | grep:21 | \ grep<模拟攻击客户机的IP>| wc -l'    # 实时查看模拟攻击客户机建立起来的连接数
watch -d 'ls -l|grep scf'         # 监测当前目录中 scf' 的文件的变化
watch -n 10 'cat /proc/loadavg'   # 10秒一次输出系统的平均负载
```
切换终端：`Ctrl+x`
退出watch：`Ctrl+g`


## Hadoop
调用文件系统(FS)Shell命令应使用 bin/hadoop fs <args>的形式。 所有的的FS shell命令使用URI路径作为参数。URI格式是scheme://authority/path。对HDFS文件系统，scheme是hdfs，对本地文件系统，scheme是file。其中scheme和authority参数都是可选的，如果未加指定，就会使用配置中指定的默认scheme。一个HDFS文件或目录比如/parent/child可以表示成hdfs://namenode:namenodeport/parent/child，或者更简单的/parent/child（假设你配置文件中的默认值是namenode:namenodeport）。大多数FS Shell命令的行为和对应的Unix Shell命令类似，不同之处会在下面介绍各命令使用详情时指出。出错信息会输出到stderr，其他信息输出到stdout。

### cat
#### 使用方法
`hadoop fs -cat URI [URI …]`: 将路径指定文件的内容输出到stdout。
### 示例
```bash
hadoop fs -cat hdfs://host1:port1/file1 hdfs://host2:port2/file2
hadoop fs -cat file:///file3 /user/hadoop/file4
```
#### 返回值
成功返回0，失败返回-1。

### chgrp
#### 使用方法
`hadoop fs -chgrp [-R] GROUP URI [URI …]`:改变文件所属的组。使用-R将使改变在目录结构下递归进行。命令的使用者必须是文件的所有者或者超级用户。

### chmod
#### 使用方法
`hadoop fs -chmod [-R] <MODE[,MODE]... | OCTALMODE> URI [URI …]`
改变文件的权限。使用-R将使改变在目录结构下递归进行。命令的使用者必须是文件的所有者或者超级用户。

### chown
#### 使用方法
`hadoop fs -chown [-R] [OWNER][:[GROUP]] URI [URI ]`
改变文件的拥有者。使用-R将使改变在目录结构下递归进行。命令的使用者必须是超级用户。

### copyFromLocal
#### 使用方法
`hadoop fs -copyFromLocal <localsrc> URI`
除了限定源路径是一个本地文件外，和put命令相似。

### copyToLocal
#### 使用方法
`hadoop fs -copyToLocal [-ignorecrc] [-crc] URI <localdst>`
除了限定目标路径是一个本地文件外，和get命令类似。

### cp
#### 使用方法
`hadoop fs -cp URI [URI …] <dest>`
将文件从源路径复制到目标路径。这个命令允许有多个源路径，此时目标路径必须是一个目录。 
#### 示例
```bash
hadoop fs -cp /user/hadoop/file1 /user/hadoop/file2
hadoop fs -cp /user/hadoop/file1 /user/hadoop/file2 /user/hadoop/dir
```
#### 返回值
成功返回0，失败返回-1。

### du
#### 使用方法
`hadoop fs -du URI [URI …]`
显示目录中所有文件的大小，或者当只指定一个文件时，显示此文件的大小。
#### 示例
`hadoop fs -du /user/hadoop/dir1 /user/hadoop/file1 hdfs://host:port/user/hadoop/dir1`
#### 返回值
成功返回0，失败返回-1。 

### dus
#### 使用方法
`hadoop fs -dus <args>`
显示文件的大小。

### expunge
#### 使用方法
`hadoop fs -expunge`
清空回收站。请参考HDFS设计文档以获取更多关于回收站特性的信息。

### get
#### 使用方法
`hadoop fs -get [-ignorecrc] [-crc] <src> <localdst> `
复制文件到本地文件系统。可用-ignorecrc选项复制CRC校验失败的文件。使用-crc选项复制文件以及CRC信息。
#### 示例
```bash
hadoop fs -get /user/hadoop/file localfile
hadoop fs -get hdfs://host:port/user/hadoop/file localfile
```
#### 返回值
成功返回0，失败返回-1。

### getmerge
#### 使用方法
`hadoop fs -getmerge <src> <localdst> [addnl]`
接受一个源目录和一个目标文件作为输入，并且将源目录中所有的文件连接成本地目标文件。addnl是可选的，用于指定在每个文件结尾添加一个换行符。

### ls
#### 使用方法
`hadoop fs -ls <args>`
如果是文件，则按照如下格式返回文件信息：
文件名 <副本数> 文件大小 修改日期 修改时间 权限 用户ID 组ID 
如果是目录，则返回它直接子文件的一个列表，就像在Unix中一样。目录返回列表的信息如下：
目录名 \<dir\> 修改日期 修改时间 权限 用户ID 组ID 
#### 示例
`hadoop fs -ls /user/hadoop/file1 /user/hadoop/file2 hdfs://host:port/user/hadoop/dir1 /nonexistentfile`
#### 返回值
成功返回0，失败返回-1。

### lsr
#### 使用方法
`hadoop fs -lsr <args>`
ls命令的递归版本。类似于Unix中的ls -R。

### mkdir
#### 使用方法
`hadoop fs -mkdir <paths> `
接受路径指定的uri作为参数，创建这些目录。其行为类似于Unix的mkdir -p，它会创建路径中的各级父目录。
#### 示例
```bash 
hadoop fs -mkdir /user/hadoop/dir1 /user/hadoop/dir2
hadoop fs -mkdir hdfs://host1:port1/user/hadoop/dir hdfs://host2:port2/user/hadoop/dir
```
#### 返回值
成功返回0，失败返回-1。

### movefromLocal
#### 使用方法
`dfs -moveFromLocal <src> <dst>`
输出一个”not implemented“信息。

### mv
#### 使用方法
`hadoop fs -mv URI [URI …] <dest>`
将文件从源路径移动到目标路径。这个命令允许有多个源路径，此时目标路径必须是一个目录。不允许在不同的文件系统间移动文件。
#### 示例
```bash
hadoop fs -mv /user/hadoop/file1 /user/hadoop/file2
hadoop fs -mv hdfs://host:port/file1 hdfs://host:port/file2 hdfs://host:port/file3 hdfs://host:port/dir1
```
#### 返回值
成功返回0，失败返回-1。

### put
#### 使用方法
`hadoop fs -put <localsrc> ... <dst>`
从本地文件系统中复制单个或多个源路径到目标文件系统。也支持从标准输入中读取输入写入目标文件系统。
####  示例
```bash
hadoop fs -put localfile /user/hadoop/hadoopfile
hadoop fs -put localfile1 localfile2 /user/hadoop/hadoopdir
hadoop fs -put localfile hdfs://host:port/hadoop/hadoopfile
hadoop fs -put - hdfs://host:port/hadoop/hadoopfile 
```
从标准输入中读取输入。
#### 返回值
成功返回0，失败返回-1。

### rm
#### 使用方法
`hadoop fs -rm URI [URI …]`
删除指定的文件。只删除非空目录和文件。请参考rmr命令了解递归删除。
#### 示例
```bash
hadoop fs -rm hdfs://host:port/file /user/hadoop/emptydir
```
#### 返回值
成功返回0，失败返回-1。

### rmr
#### 使用方法
`hadoop fs -rmr URI [URI …]`
delete的递归版本。
#### 示例
```bash
hadoop fs -rmr /user/hadoop/dir
hadoop fs -rmr hdfs://host:port/user/hadoop/dir
```
#### 返回值
成功返回0，失败返回-1。

### setrep
#### 使用方法
`hadoop fs -setrep [-R] <path>`
改变一个文件的副本系数。-R选项用于递归改变目录下所有文件的副本系数。
#### 示例
```bash
hadoop fs -setrep -w 3 -R /user/hadoop/dir1
```
#### 返回值
成功返回0，失败返回-1。

### stat
#### 使用方法
`hadoop fs -stat URI [URI …]`
返回指定路径的统计信息。
#### 示例：
`hadoop fs -stat path`
#### 返回值
成功返回0，失败返回-1。

### tail
#### 使用方法
`hadoop fs -tail [-f] URI`
将文件尾部1K字节的内容输出到stdout。支持-f选项，行为和Unix中一致。
#### 示例
`hadoop fs -tail pathname`
#### 返回值
成功返回0，失败返回-1。

### test
#### 使用方法
`hadoop fs -test -[ezd] URI`
#### 选项：
*   -e 检查文件是否存在。如果存在则返回0。
*   -z 检查文件是否是0字节。如果是则返回0。 
*   -d 如果路径是个目录，则返回1，否则返回0。

#### 示例：
`hadoop fs -test -e filename`

### text
#### 使用方法
`hadoop fs -text <src>`
将源文件输出为文本格式。允许的格式是zip和TextRecordInputStream。

### touchz
#### 使用方法
`hadoop fs -touchz URI [URI …]`
创建一个0字节的空文件。
#### 示例
`hadoop -touchz pathname`
#### 返回值
成功返回0，失败返回-1。


## Spark


## Kafka
1.  新建主题
    <br>`bin/kafka-topics.sh --create --zookeeper master:2181,slave01:2181,slave02:2181/kafka --replication-factor 2 --partitions 2 --topic flume2kafka`
2.  查看主题
    <br>`bin/kafka-topics.sh --describe --zookeeper master:2181 --topic flume2kafka`
3.  查看所有主题
    ```bash
    bin/kafka-topics.sh --list --zookeeper master:2181,slave01:2181,slave02:2181
    bin/kafka-topics.sh --list --zookeeper master:2181,slave01:2181,slave02:2181/kafka
    ```
4.  改变主题的分区数量
    <br>`bin/kafka-topics.sh --alter --zookeeper master:2181 --topic flume2kafka --partitions 4`
5.  增删改主题的配置参数
    ```bash
    bin/kafka-topics.sh --alter --zookeeper master:2181 --topic flume2kafka--config key=value
    bin/kafka-topics.sh --alter --zookeeper master:2181 --topic flume2kafka--deleteConfig key
    ```
6.  启动生产者以发送消息
    ```bash
    bin/kafka-console-producer.sh --broker-list master:9092,slave01:9092 --topic flume2kafka
    bin/kafka-console-producer.sh --broker-list master:9092 --topic flume2kafka < /mnt/hgfs/vmshares/access_log.txt
    ```
7.  启动消费者以接收消息
    <br>`bin/kafka-console-consumer.sh --bootstrap-server master:9092,slave01:9092,slave02:9092  --topic flume2kafka --from-beginning`
8.  执行leader分布的再平衡
    <br>`bin/kafka-preferred-replica-election.sh --zookeeper master:2181`
    Kafka会自动做leader分布再平衡，但不会发现问题即刻就做，有延迟。
9.  删除主题
    <br>`bin/kafka-topics.sh --delete --zookeeper master:2181  --topic flume2kafka`