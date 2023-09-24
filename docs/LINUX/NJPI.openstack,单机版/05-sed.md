# sed批量修改文件

```shell
sed -i.bak -r 'pattern' filename
-i  修改源文件
-i.bak  update and backup
-r 正则表达式 pattern
sed -i.bak -r 's/(module-store = )direct/\1undirect/g' semanamge.conf
s subtiture 替换
() 分组
s/suorce/dest/g
s/module-store = direct/module-store = undirect/g

sudo sed -i -r 's/(expand-check)(=)0/\1\21/g' semanage.conf
说明： 
-r: 正则表达式 pattern
/c: 替换
(分组)
0/: 整行替换
\1: 分组的第一个括号
\2: 分组的第二个括号
/g: 结束
sed -i -r 's/(SELINUX=)0/\1enable/g'
```



查看虚拟机是否支持**虚拟化**：`grep -E 'svm|vmx' /proc/cpuinfo | grep nx`

停止`NetworkManager`：`systemctl disable NetworkManager.service`

`systemctl stop NetworkManager.service`

