# grep的使用
##  1、grep作用 
grep在文本文件中搜索指定的字符串
##  2、grep语
```shell
grep [OPTIONS] PATTERN [FILE...]
```
说明：
options:
> -v  翻转显示
> -i  忽略搜索字符串大小写
> -o 只匹配搜索到的字符串
> -r 递归搜索
> -c 统计匹配的行数
> -A n 显示匹配之后n行
> -B n 显示匹配之前n行
> -C n显示匹配前后n行
>  -l 只显示文件名
## 3、应用举例
### 3.1 查找当前文件夹下，包含this的文件名
```
[root@controller ~]# grep -l "this" *
answer-file
grep_demo_file1.txt
grep_demo_file.txt
uninstall2.sh
```
### 3.2显示匹配的行
```
[root@controller ~]# grep -c "this" grep_demo_file.txt
3
```
### 3.3显示不以this开头的内容
```
[root@controller ~]# grep -i -v "^this" grep_demo_file.txt

Two lines above this line is empty.
And this is the last line.
```

### 3.4搜索整is是个单词 ,而不是某个词的一部分

``` shell
grep -w "is" test.txt
```


### 3.5只显示搜索到的内容

``` 
[root@controller ~]# grep -w -o "is" grep_demo_file.txt
is
is
is
```
