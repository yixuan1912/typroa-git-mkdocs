[TOC]

### scp

```BASH
-q:  quiet     不显示传输进度条。
-r:   recuersive    递归的
-p:   properity    保留属性
-v:   verbose    显示详细输出
-i:   identity_file      从指定文件中读取传输时使用的密钥文件，此参数直接传递给ssh 
-P:  port 		端口（大写P）
# scp file user@host:file    发送文件给别人
# scp user@host:file         从别人下载文件到自己
```

### Parted

```BASH
帮助选项：
-h, --help                    显示此求助信息 
-l, --list                    列出所有设别的分区信息
-i, --interactive             在必要时，提示用户 
-s, --script                  从不提示用户 
-v, --version                 显示版本

操作命令：
cp [FROM-DEVICE] FROM-MINOR TO-MINOR           #将文件系统复制到另一个分区 
help [COMMAND]                                 #打印通用求助信息，或关于 COMMAND 的信息 
mklabel 标签类型                               #创建新的磁盘标签 (分区表) 
mkfs MINOR 文件系统类型                        #在 MINOR 创建类型为“文件系统类型”的文件系统 
mkpart 分区类型 [文件系统类型] 起始点 终止点   #创建一个分区 
mkpartfs 分区类型 文件系统类型 起始点 终止点   #创建一个带有文件系统的分区 
move MINOR 起始点 终止点                       #移动编号为 MINOR 的分区 
name MINOR 名称                                #将编号为 MINOR 的分区命名为“名称” 
print [MINOR]                                  #打印分区表，或者分区 
quit                                           #退出程序 
rescue 起始点 终止点                           #挽救临近“起始点”、“终止点”的遗失的分区 
resize MINOR 起始点 终止点                     #改变位于编号为 MINOR 的分区中文件系统的大小 
rm MINOR                                       #删除编号为 MINOR 的分区 
select 设备                                    #选择要编辑的设备 
set MINOR 标志 状态                            #改变编号为 MINOR 的分区的标志
```

```BASH
1、选择分区硬盘
首先类似fdisk一样，先选择要分区的硬盘，此处为/dev/hdd： ((parted)表示在parted中输入的命令，其他为自动打印的信息)

[root@10.10.90.97 ~]# parted /dev/hdd
GNU Parted 1.8.1
Using /dev/hdd
Welcome to GNU Parted! Type 'help' to view a list of commands.
2、创建分区
选择了/dev/hdd作为我们操作的磁盘，接下来需要创建一个分区表(在parted中可以使用help命令打印帮助信息)：

(parted) mklabel
New disk label type? gpt    (我们要正确分区大于2TB的磁盘，应该使用gpt方式的分区表，输入gpt后回车)
3、完成分区操作
创建好分区表以后，接下来就可以进行分区操作了，执行mkpart命令，分别输入分区名称，文件系统和分区 的起止位置

(parted) mkpart
Partition name? []? dp1
File system type? [ext2]? ext3
Start? 0           （可以用百分比表示，比如Start? 0% , End? 50%）
End? 500GB
4、验证分区信息
分好区后可以使用print命令打印分区信息，下面是一个print的样例

(parted) print
Model: VBOX HARDDISK (ide)
Disk /dev/hdd: 2199GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Number Start End Size File system Name Flags
1 17.4kB 500GB 500GB dp1
5、删除分区示例
如果分区错了，可以使用rm命令删除分区，比如我们要删除上面的分区，然后打印删除后的结果

(parted)rm 1               #rm后面使用分区的号码，就是用print打印出来的Number
(parted) print
Model: VBOX HARDDISK (ide)
Disk /dev/hdd: 2199GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Number Start End Size File system Name Flags
6、完整示例
按照上面的方法把整个硬盘都分好区，下面是一个分完后的样例

(parted) mkpart
Partition name? []? dp1
File system type? [ext2]? ext3
Start? 0
End? 500GB
(parted) mkpart
Partition name? []? dp2
File system type? [ext2]? ext3
Start? 500GB
End? 2199GB
(parted) print
Model: VBOX HARDDISK (ide)
Disk /dev/hdd: 2199GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Number Start End Size File system Name Flags
1 17.4kB 500GB 500GB dp1
2 500GB 2199GB 1699GB dp2
7、格式化操作
完成以后我们可以使用quit命令退出parted并使用系统的mkfs命令对分区进行格式化了。

[root@10.10.90.97 ~]# fdisk -l
WARNING: GPT (GUID Partition Table) detected on '/dev/hdd'! The util fdisk doesn't support GPT. Use GNU Parted.
Disk /dev/hdd: 2199.0 GB, 2199022206976 bytes
255 heads, 63 sectors/track, 267349 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Device Boot Start End Blocks Id System
/dev/hdd1 1 267350 2147482623+ ee EFI GPT
[root@10.10.90.97 ~]# mkfs.ext3 /dev/hdd1
[root@10.10.90.97 ~]# mkfs.ext3 /dev/hdd2
[root@10.10.90.97 ~]# mkdir /dp1 /dp2
[root@10.10.90.97 ~]# mount /dev/hdd1 /dp1
[root@10.10.90.97 ~]# mount /dev/hdd2 /dp2
```

###  tr替换字符串等

- ```BASH
  用法：tr [-cdst]... SET1 [SET2]
  从标准输入中替换、缩减和/或删除字符，并将结果写到标准输出。
  
  -c, --complement：反选设定字符。也就是符合 SET1 的部份不做处理，不符合的剩余部份才进行转换为 SET2
  -d, --delete：删除指令字符
  -s, --squeeze-repeats：缩减连续重复的字符成指定的单个字符
  -t, --truncate-set1：削减 SET1 指定范围，使之与 SET2 设定长度相等
  --help：显示程序用法信息
  --version：显示程序本身的版本信息
  
  SET 是一组字符串，一般都可按照字面含义理解。解析序列如下：
  \NNN  八进制值为NNN 的字符(1 至3 个数位)
    \\            反斜杠
    \a            终端鸣响
    \b            退格
    \f            换页
    \n            换行
    \r            回车
    \t            水平制表符
    \v            垂直制表符
    字符1-字符2   从字符1 到字符2 的升序递增过程中经历的所有字符
    [字符*]       在SET2 中适用，指定字符会被连续复制直到吻合设置1 的长度
    [字符*次数]   对字符执行指定次数的复制，若次数以 0 开头则被视为八进制数
    [:alnum:]     所有的字母和数字
    [:alpha:]     所有的字母
    [:blank:]     所有呈水平排列的空白字符
    [:cntrl:]     所有的控制字符
    [:digit:]     所有的数字
    [:graph:]     所有的可打印字符，不包括空格
    [:lower:]     所有的小写字母
    [:print:]     所有的可打印字符，包括空格
    [:punct:]     所有的标点字符
    [:space:]     所有呈水平或垂直排列的空白字符
    [:upper:]     所有的大写字母
    [:xdigit:]    所有的十六进制数
    [=字符=]      所有和指定字符相等的字符
  
  1.替换字符串
  # seq 9 |tr "\n" ","
  1,2,3,4,5,6,7,8,9,
  
  2.删除多余的空行
  # cat last.txt | tr -s '\n'
  
  3.删除指定的字符串
  # cat test_openrc.sh |tr -d '#'    			 删除了#
  # cat test_openrc.sh |tr -d '[a-z]#[0-9]'    删除了小写字母、#和数字
  
  4.大小写转换
  # cat  test_openrc.sh |tr '[A-Z]'  '[a-z]'   大写转换为小写
  
  5.字符串去重
  # cat  test_openrc.sh |tr -s '[0-9][A-Z]'    把连续的重复的数字和大写字母只保留了一个
  
  6.反向选择替换
  # cat  test_openrc.sh |tr -c '[0-9]' ' '     把除了数字之外的字符串换成空格
  # cat  test_openrc.sh |tr -c '[0-9]' ' ' |tr -d ' '   上面的命令+删掉空格
  ```



### blkid显示块设备属性查看硬盘UUID号

#### 功能描述

使用blkid命令可以用来查询系统的块设备（包括交换分区）所使用的文件系统类型、卷标、UUD等信息。　　　　

在Linux下可以使用blkid命令对查询设备上所采用文件系统类型进行查询。blkid主要用来对系统的块设备(包括交换分区)所使用的文件系统类型、LABEL、UUID等信息进行查询。

#### UUID作用

　　UUID是一个标识你系统中的存储设备的字符串，其目的是帮助使用者唯一的确定系统中的所有存储设备，不管它们是什么类型的。它可以标识DVD驱动器，USB存储设备以及你系统中的硬盘设备等。

特点：

它是真正的唯一标志符

Linux中的许多关键功能现在开始依赖于UUID

UUID号：分区必须格式化后才会有UUID号。

#### **命令语法：**

blkid 分区设备

#### **选项含义：**

| **选项** | **含义**                                                     |
| -------- | ------------------------------------------------------------ |
| -L<卷标> | 卷标转换为设备名                                             |
| -U<UUID> | UUID转换为设备名                                             |
| -p       | 探测低级别的超级块                                           |
| -i       | 收集有关I/O限制的信息                                        |
| -o<格式> | 按以下列输出格式 value:显示标签的值 device:只显示设备的名称 list:以用户友好的格式显示设备 udev：以“键=值”的方式显示，方便导入udev的环境 export:以“键=值”的方式显示，方便导入环境 full:显示所有标签 |
| -h       | 显示帮助信息                                                 |

#### 示例：

查看磁盘分区/dev/sda3的文件系统类型、卷标、UUID等信息

```
[root@localhost ~]# blkid /dev/sda3

/dev/sda3: UUID="e7adfeb9-8749-48cd-90cd-61de56c9af74" TYPE="ext4"
```


加入管道符查找UUID号

```
[root@xuegod163 ~]# blkid | grep sdb1

/dev/sdb1: LABEL="cc" UUID="0e77cda6-4c43-47a9-ae7e-c27661234760" TYPE="ext4"

[root@xuegod163 ~]# tune2fs -l /dev/sdb1 | grep UUID

Filesystem UUID: 0e77cda6-4c43-47a9-ae7e-c27661234760
```


输出到自动挂载/etc/fstab配置文件

```
[root@localhost ~]# blkid /dev/sdb3 >> /etc/fstab

[root@xuegod163 ~]# vim /etc/fstab

UUID=0e77cda6-4c43-47a9-ae7e-c27661234760 /sdb1 ext4 defaults 0 0
```

查看UUID是83ff032a-757e-4466-8d64-0885281ce22b的设备名

```
[root@localhost ~]# blkid -U 83ff032a-757e-4466-8d64-0885281ce22b
/dev/sda1
```

查看卷标是boot的设备名

```
[root@localhost ~]# blkid -L boot
```

查看磁盘分区/dev/sda1低级别的超级块信息

```
[root@localhost ~]# blkid -p /dev/sda1
/dev/sda1: UUID="83ff032a-757e-4466-8d64-0885281ce22b" TYPE="xfs" USAGE="filesystem" PART_ENTRY_SCHEME="dos" PART_ENTRY_TYPE="0x83" PART_ENTRY_FLAGS="0x80" PART_ENTRY_NUMBER="1" PART_ENTRY_OFFSET="2048" PART_ENTRY_SIZE="2097152" PART_ENTRY_DISK="8:0" 
```

收集磁盘分区/dev/sda1有关I/O限制的信息

```
[root@localhost ~]# blkid -i /dev/sda1
DEVNAME=/dev/sda1
MINIMUM_IO_SIZE=512
PHYSICAL_SECTOR_SIZE=512
LOGICAL_SECTOR_SIZE=512
```

显示磁盘所有标签

```
[root@localhost ~]# blkid -o full
/dev/mapper/centos-root: UUID="f74ee8de-84a2-49cb-82e3-1c5fef2cd92b" TYPE="xfs" 
/dev/sda2: UUID="2FqB5r-teLk-miR5-c46S-ceCy-ySxa-K1hk17" TYPE="LVM2_member" 
/dev/sda1: UUID="83ff032a-757e-4466-8d64-0885281ce22b" TYPE="xfs" 
/dev/mapper/centos-swap: UUID="274d2ea0-deba-4be9-9f32-c080efd68096" TYPE="swap" 
```

查看所有磁盘分区的文件系统类型、卷标、UUID等信息

```
[root@localhost ~]# blkid
/dev/mapper/centos-root: UUID="f74ee8de-84a2-49cb-82e3-1c5fef2cd92b" TYPE="xfs" 
/dev/sda2: UUID="2FqB5r-teLk-miR5-c46S-ceCy-ySxa-K1hk17" TYPE="LVM2_member" 
/dev/sda1: UUID="83ff032a-757e-4466-8d64-0885281ce22b" TYPE="xfs" 
/dev/mapper/centos-swap: UUID="274d2ea0-deba-4be9-9f32-c080efd68096" TYPE="swap" 
```

查看所有磁盘分区的UUID

```
[root@localhost ~]# blkid -s UUID
/dev/mapper/centos-root: UUID="f74ee8de-84a2-49cb-82e3-1c5fef2cd92b" 
/dev/sda2: UUID="2FqB5r-teLk-miR5-c46S-ceCy-ySxa-K1hk17" 
/dev/sda1: UUID="83ff032a-757e-4466-8d64-0885281ce22b" 
/dev/mapper/centos-swap: UUID="274d2ea0-deba-4be9-9f32-c080efd68096" 
```

查看/dev/sda1磁盘分区的UUID

```
[root@localhost ~]# blkid -s UUID /dev/sda1
/dev/sda1: UUID="83ff032a-757e-4466-8d64-0885281ce22b" 
```



