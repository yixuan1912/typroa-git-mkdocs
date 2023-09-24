# find命令详解
## 1、find命令的作用
是对文件或者目录进行查找操作的函数
## 2、find语法
```shell
find [option] [PATH] [expression]
```
说明：
1. [option] 参数说明：-H，-L，-P 是否处理符号链接文件
2. [PATH] 需要查找的路径
3. [expression] 查找表达式
## 3  expression
expression由三部分组成：
1. options，所有的参数均返回true,如：-maxdepth，-mindepth
2. tests, 如-ctime，-iname，-perm，-type,
3. actions
### 3.1 表达式option选项
#### 3.1.1 -maxdepth , -mindepth
按深度进行查找：
1. -maxdepth 设置最大深度
2. -mindepth 设置最小深度
#### 3. 1.2 -help
显示帮助
#### 3.1.3 -version
显示版本
#### 3.1.4 -noleaf
关闭优化
### 3.2 表达式test选项
#### 3.2.1  -name
 按basename文件名进行搜索，basename中不能有/，文件名要有引号括起来
#### 3.2.2 -path
 按文件全路径：dirname+basename进行搜索，文件名最好有引号扩起来。在路径中可以有/号
#### 3.2.3 -type
 根据文件类型来搜索，参数如下表所示：
 |参数名称|参数功能|
 |--|--|
 |b |     block (buffered) special|
 | c|      character (unbuffered) special|
 |d |     directory|
 |p  |    named pipe (FIFO)|
|f   |   regular file|
|l    |  symbolic link; this is never true if the -L option or the follow option is in effect, unless the symbolic link  is broken.  If you want to search for symbolic links when -L is in effect, use -xtype.|
|s     | socket|
|D     | door (Solaris)|
#### 3.2.4 -perm
按文件的权限进行查找
1. octal（八进制）方式进行查找
   应用举例：
       * 777 查找文件权限为777的文件
       * -777查找文件所有者，文件所属组，其他用户权限为7的文件
       * /777查找文件所有者，文件所属组，其他用户权限为7的文件
2. ugo/rwx方式进行查找
   u--user
   g--group
   o-other
   r-read
   w-write
   x-execute
   -o=rw，查找其他用户具有读写权限的文件
   -o=-w-r,查找其他用户没有读写权限的文件
#### 3.2.5 -size
按文件大小进行查找
1. -size 200k,  查找大小为200k的文件
2. -size -200k，查找大小不超过200k的文件
3. -size +200k, 查找大小超过200k的文件
#### 3.2.6 -user 
按文件属主进行查找
 -user lqj 
 可以同时查找多个用户

### 3.2.7 -group
按文件所属的组进行查找
可以同时查找多个组，同-user
### 3.3 表达式actions选项
#### 3.3.1 -print
打印功能：
 1. 默认以\n将找到的文件进行分隔。
 2. -print0 使用\0进行文件分隔
 3. -printf，同C语言printf选项
#### 3.3.2 -delete
删除文件
```shell
[root@controller ~]# ls
anaconda-ks.cfg      grep_demo_file1.txt.bak  keystonerc_admin  uninstall2.sh
answer-file          grep_demo_file.txt       test.txt          uninstall.sh
grep_demo_file1.txt  grep_demo_file.txt.bak   test.txt.bak
[root@controller ~]# find . -name "*.bak" -delete
[root@controller ~]# ls
anaconda-ks.cfg  grep_demo_file1.txt  keystonerc_admin  uninstall2.sh
answer-file      grep_demo_file.txt   test.txt          uninstall.sh
```
#### 3.3.3 -exec command ;
执行命令
**注意：**
以分号结尾，有的命令行中需要进行转义即为：\;
备份.txt文件为.bak文件
```shell
root@controller ~]# ls
anaconda-ks.cfg  grep_demo_file1.txt  keystonerc_admin  uninstall2.sh
answer-file      grep_demo_file.txt   test.txt          uninstall.sh
[root@controller ~]#
[root@controller ~]# find . -name "*.txt"  -exec cp {} {}.bak \;
[root@controller ~]# ls
anaconda-ks.cfg      grep_demo_file1.txt.bak  keystonerc_admin  uninstall2.sh
answer-file          grep_demo_file.txt       test.txt          uninstall.sh
grep_demo_file1.txt  grep_demo_file.txt.bak   test.txt.bak
[root@controller ~]#
```
#### 3.3.4 -ok command ;
使用方法同-exec command ；执行前需要询问用户是否执行？
```shell
[root@controller ~]# find . -name "*.txt"  -ok cp {} {}.bak \;
< cp ... ./test.txt > ? y
< cp ... ./grep_demo_file.txt > ? y
< cp ... ./grep_demo_file1.txt > ? y
[root@controller ~]# ls
anaconda-ks.cfg      grep_demo_file1.txt.bak  keystonerc_admin  uninstall2.sh
answer-file          grep_demo_file.txt       test.txt          uninstall.sh
grep_demo_file1.txt  grep_demo_file.txt.bak   test.txt.bak
```
#### 3.3.5 -ls 
浏览文件
```shell
find . -name "*.txt"  -ls
204978657    4 -rw-r--r--   1 root     root           52 10月  2 13:11 ./test.txt
204978661    4 -rw-r--r--   1 root     root          233 10月  2 13:49 ./grep_demo_file.txt
204978656    4 -rw-r--r--   1 root     root          233 10月  2 13:50 ./grep_demo_file1.txt
```
## 4、 关系
### 4.1 非
 1. ！
 2. -not
### 4.2 与
1. -a
2. expr1 expr2
### 4.3 或
 -o
## 5、find -exec command ; 与 xargs的区别
### 5.1 find exec执行
  1. exec是每处理一个文件/目录，都要启动一次命令，效率不好 。
  2. 格式麻烦，必须用 {} 做文件的代位符，必须用 \; 作为命令的结束符，书写不便。
### 5.2 xargs
 1. xargs不能操作文件名有空格的文件。
 2. 一次可以处理多个文件。
 3. 不需要以；结尾。
 4. 不强制使用{}

## 6、应用举例
### 6.1查找当前目录，给出找到文件的类型
```shell
find . -type f -exec file '{}' \;
```
###  6.2 按权限查找
```shell
find . -perm /220
find . -perm /u+w,g+w
find . -perm /u=w,g=w
```