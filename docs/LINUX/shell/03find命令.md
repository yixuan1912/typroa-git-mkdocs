## find命令

>  find: 用来在指定目录下查找文件
>
> **格式：**find  [搜索范围]     [ 匹配条件]
>
> `find pathname -options [-print -exec -ok....]`

- `-name` 根据文件名搜索
- `-iname` 在搜索时不区分大小写(可使用通配符`*  ？`任意/单个字符)

- `-size n`    查找文件大小为n的**块**
  - `+n`  大于     `-n`小于      `n`等于
    - find   /   -size    +204800     //查找大于100M的文件
    - **1数据块=512字节=0.5K**
- `-a`(and)表示两个条件同时满足
  
  - find  /  -size   +102400 -a -size  -201800
- `-o`(or)表示满足其中一个就行
  
- find  /  -size   -102400   -o  -size  +201800
  
- `-type` 根据文件类型查找
  - `f`文件   `d`目录    `l`软链接文件
    - find /etc  -name  init* -a -type  l

- `-inum`   根据i节点查找    硬链接
  
- `find  .  -inum  节点号` 每个文件都有一个节点号，但一个节点号可能对应多个文件
  
- `-user` 根据所有者来查找

- `-group`   所属组
  
- find  /home   -user  lhq
  
- 根据**时间**属性查找
  - `-atime  n`访问时间（access）`+n`天前被访问过的文件，`-n`没超过n天前被访问的文件
  - `-ctime`   文件属性 change 检查文件索引节点被修改的时间
  - `-mtime`   文件内容modify 检查文件内容被修改的时间

- `-perm n`   查找权限为**n**的文件
  
- find  .  -perm  644
  
- 对查找的结果做指定的操作

  - `-exec  命令   { }  \;`  对比配指定条件的文件执行命令中的操作。

    `-ok  命令   { }  \;`    同上，只是添加用户确认

    - find /etc -name init* -a -type d -exec ls -dl {} \;  （寻找etc下以init开头的文件夹，并显示其详细信息）



