# sed的使用
## 1、sed简介
1. sed是一种流编辑器，可以配合正则表达式使用，处理文本文件非常方便，在处理时，把当前处理的行存储在临时缓冲区内，称为“模式空间”，使用sed命令处理缓冲区中的内容，处理完成后把缓冲区的内容送到屏幕，接着处理下一行，重复不断，直到文件结束。
2. sed处理文件时源文件内容不会发生改变，除非使用写命令。
## 2、sed 的使用
### 2.1语法格式
sed [options] 'command' file(s)
说明：
> 1. options常用参数说明
> -  `-i` 更新操作，`-i.bak`对源文件  备份和更新
> -  -r 使用扩展的正则表达式
>-  -e 多点编辑，可以在一行中执行多条命令
> - -V显示版本号
> - -n只显示匹配的行
> 2. Command

> | 命令 |                                   功能                                   |
> | ---- | ----------------------------------------------------------------------- |
> | a\   | 在当前行后添加一行或多行。多行时除最后一行外，每行末尾需用“\n”续行          |
> | c\   | 用此符号后的新文本替换当前行中的文本。多行时除最后一行外，每行末尾需用"\n"续行 |
>| i\   | 在当前行之前插入文本。多行时除最后一行外，每行末尾需用"\n"续行                |
> | d    | 删除行                                                                   |
> | h    | 把模式空间里的内容复制到暂存缓冲区                                          |
> | H    | 把模式空间里的内容追加到暂存缓冲区                                          |
> | g    | 把暂存缓冲区里的内容复制到模式空间，覆盖原有的内容                           |
> | G    | 把暂存缓冲区的内容追加到模式空间里，追加在原有内容的后面                     |
> | l    | 列出非打印字符                                                            |
> | p    | 打印行                                                                   |
> | n    | 读入下一输入行，并从下一条命令而不是第一条命令开始对其的处理                  |
> | q    | 结束或退出sed                                                             |
> | r    | 从文件中读取输入行                                                        |
> | !    | 对所选行以外的所有行应用命令                                               |
> | s    | 用一个字符串替换另一个                                                     |
> | g    | 在行内进行全局替换                                                        |
> | w    | 将所选的行写入文件                                                        |
> | x    | 交换暂存缓冲区与模式空间的内容                                             |
> | y    | 将字符替换为另一字符（不能对正则表达式使用y命令）                            |
### 2.2 地址格式
1.  number 只匹配指定的number行
2. first~step，给出起始行，给出步长，进行匹配 1~2匹配奇数行，2~2匹配偶数行
3. /regex/，匹配正则表达式匹配的行。
4. 0,addr2 表示范围，0表示开始行，addr2表示结束行
5. $ 表示尾行
6. addr1,+N,表示从addr1开始及其后的N行
7. addr1,~N,表示从addr1开始，直到行数是N的整数倍
### 2.3 sed定界符
sed默认使用/作为定界符，当定界符出现在字符串内部时，需要进行转义，\ /
### 2.4 &
已匹配字符串标记&
```shell
sed 's/^192.168.0.1/&localhost/' file
```
运行结果：
```shell
192.168.0.1localhost
```
### 2.5 应用示例
1. 示例文件:test.txt
``` text
this is a test! 
this is a test2!
lqj
zhangsan
lisi
   wangwu
   zhaoliu
red
blue
green
```
2. 在this is a test!之后插入1行，this is a test1!
``` shell
sed -i.bak -r '/this is a test!/athis is a test1!' test.txt
```
3. 把this is a test!替换为this is a test3！
```shell
  sed -i.bak -r '/this is a test!/cthis is a test3!' test.txt
```
4. 把wangwu替换为zhaoliu
```shell
sed -i.bak -r 's/wanglu/zhaoliu' test.txt
```
5. 打印文件的奇数行
```shell
sed  -n "p;n" test.txt
```
6. 打印文件的偶数行

```shell
sed -n "n;p" text.txt
```
7.  删除1到5行，同时把my替换为MY（-e的使用）
```shell
sed -e '1,5d' -e 's/my/MY/' test.txt 
```
8. 删除所有的空行与行前空格
数据文件：
```shell
my cat's name is betty

	This is your dog

my dog's name is frank

	This is your fish

my fish's name is george

	This is your goat

my goat's name is adam
```
sed命令：
```shell
  sed -i.bak  -r "/^$/d;s/^\s*//g" test.txt
```
运行结果：
```shell
my cat's name is betty
This is your dog
my dog's name is frank
This is your fish
my fish's name is george
This is your goat
my goat's name is adam
```
9. 如果my在行首被匹配，把my替换为My，移动到匹配行的下一行，替换这一行的this为This,并打印该行
数据文件：
```shell
my cat's name is betty
this is your dog
my dog's name is frank
this is your fish
my fish's name is george
this is your goat
my goat's name is adam
```
命令如下：
```shell
sed -i -r "s/^my/My/;n;s/^this/This/" test.txt
```
运行结果：
```shell
My cat's name is betty
This is your dog
My dog's name is frank
This is your fish
My fish's name is george
This is your goat
My goat's name is adam
```



10. 定义变量：

变量定义：   a=2
调用：   $变量名
      sed  -n ‘&apos;$a&apos;,5p’ 文件名
          注意在‘’中变量需要再单独引起来
   也可以用“”，变量就可以直接调取
      sed  -n  “$a,5p” 文件名

```
[root@yum ~]# a=2
[root@yum ~]# sed -n "$a,4p" cbp
```

