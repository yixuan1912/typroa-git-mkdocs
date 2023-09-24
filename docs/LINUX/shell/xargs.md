# xargs的使用
## 2.1 为什么要引入xargs？
1. linux命令可以从两个地方读取要处理的内容，一个是通过命令行参数，一个是标准输入。
2. 有些命令不能处理标准输入，如rm，kill。需要xargs把输入流向命令参数进行转换
## 2.2xargs语法
### 2.2.1基本语法
```shell
 command1 | xargs [options] command2
```
### 2.2.2执行流程: 
> 1. command1首先执行，把执行结果通过管道传递给xargs
> 2. xargs把传递过来的结果重新处理，把结果作为命令参数传递给command2
> 3. command2执行 xargs传递过来的参数
> 4. xargs处理的顺序：先分割，再分批(可以实现10个10个的传送 )，然后传递到参数位。
### 2.2.3 xargs分割语法
分割有三种方法：
1. 独立的xargs ，默认使用空格进行分割
2. xargs -d *delim* ，指定自定义分隔符
3. xargs  -0 ，分割\0分隔的字符串，通常用在print0输出
### 2.2.4 xargs分批语法
1. -n *nums* 选项,一次输出的参数个数
2. -L max-lines选项, 指定一次输入的参数的个数，与n相同，区别在于n是按照空格分隔，L按段分隔
3. -i 逻辑上使用分批结果，i可以指定多个参数位,使用{}指代分批结果
4.  -I 同i, I可以指代其他符号
### 2.2.5 xargs默认语法
如果xargs后面没有命令，默认为echo命令
### 2.2.6
## 2.3 应用举例
### 2.3.1查找当前目录下所有的txt 文件并压缩
```shell
[root@openstackallinone ~]# find . -name "*.txt"  -type f -print | xargs tar -cvzf test.tar.gz
./test.txt
./test2.txt
[root@openstackallinone ~]# ls -l
总用量 76
-rw-------. 1 root root   955 7月   1 02:39 anaconda-ks.cfg
-rw-------  1 root root 51800 9月  27 08:38 answer-file
-rw-------  1 root root   329 9月  27 08:48 keystonerc_admin
-rw-r--r--  1 root root   166 9月  30 17:14 test2.txt
-rw-r--r--  1 root root   233 9月  30 17:17 test.tar.gz
-rw-r--r--  1 root root   166 9月  30 04:47 test.txt
-rw-r--r--  1 root root   156 9月  30 04:17 test.txt.bak
```
### 2.3.2  删除查找到的日志
```shell
[root@openstackallinone ~]# find . -name "*.log" -type f -print
./mysql.log
./neutron.log
[root@openstackallinone ~]# find . -name "*.log" -type f -print0
./mysql.log./neutron.log[root@openstackallinone ~]# find . -name "*.log" -type f -print0
./mysql.log./neutron.log[root@openstackallinone ~]# find . -name "*.log" -type f -print0|xargs -0 rm -f
[root@openstackallinone ~]# ls
anaconda-ks.cfg  keystonerc_admin  test.tar.gz  test.txt.bak
answer-file      test2.txt         test.txt
```
**说明：**
1. find print 默认以\n进行分隔，即打印多行
2. find print0 以\0代替\n，即不分行打印，响应地xargs 要以 -0进行分隔
###  2.3.3 查找当前目录下所有用户具有读写执行权限的文件，并收回读写权限
```shell
[root@openstackallinone ~]# find -perm "-o=rw" -type f -print0 | xargs -0 chmod  "o-r-w"

读 写 执行
421

[root@openstackallinone ~]# ls -l
总用量 76
-rw-------. 1 root root   955 7月   1 02:39 anaconda-ks.cfg
-rw-------  1 root root 51800 9月  27 08:38 answer-file
-rw-------  1 root root   329 9月  27 08:48 keystonerc_admin
-rw-r--r--  1 root root   166 9月  30 17:14 test2.txt
-rwxrwx--x  1 root root   233 9月  30 17:17 test.tar.gz
-rwxrwx--x  1 root root   166 9月  30 04:47 test.txt
-rwxrwx--x  1 root root   156 9月  30 04:17 test.txt.bak
```
**说明：**
1.  find -perm "octal "/ "-ugo=rwx" 可以使用八进制，或者使用-ugo=rwx进行匹配
2. "o-r-w" 表示o表示other,其他用户，-r表示去掉读权限，-w表示去掉写权限
### 2.3.4 批量下载
```shell
cat urls.txt | xargs wget
```
### 2.3.5批量查找并替换文本
```shell
[root@openstackallinone ~]# grep -rl "is"  --include="*.txt" ./* |xargs sed -i "s/is/Is/g"
[root@openstackallinone ~]# cat test.txt
My cat's name Is betty
ThIs Is your dog
My dog's name Is frank
ThIs Is your fIsh
My fIsh's name Is george
ThIs Is your goat
thIs Is hIs  sheep
my goat's name Is adam
```
### 2.3.6 查找没有文件名与.txt结尾的文件，增加.bak
```shell
[root@openstackallinone ~]# ls *.txt *.log a b c d | xargs -i mv {} {}.bak
[root@openstackallinone ~]# ls
a.bak            answer-file  c.bak  keystonerc_admin  my note.log.bak  test2.txt.bak  test.tar.gz
anaconda-ks.cfg  b.bak        d.bak  logdir            note.log.bak     test3.txt.bak  test.txt.bak
```
### 2.3.7查找.bak结尾的文件，去掉.bak
```shell
[root@openstackallinone ~]# ls
a.bak-.bak       c.bak-.bak        note.log.bak-.bak   test.txt.bak-.bak
anaconda-ks.cfg  d.bak-.bak        test2.txt.bak-.bak
answer-file      keystonerc_admin  test3.txt.bak-.bak
b.bak-.bak       logdir            test.tar.gz
[root@openstackallinone ~]# ls *.bak|awk '{print substr($1,1,index($1,".bak")-1)}'| xargs -i mv {}.bak-.bak {}
[root@openstackallinone ~]# ls 
a                b  keystonerc_admin  test2.txt    test.txt
anaconda-ks.cfg  c  logdir            test3.txt
answer-file      d  note.log          test.tar.gz
```

