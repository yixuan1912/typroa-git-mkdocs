### docker基本命令

- `docker ps -a `:查看进程
- `ocker run -it --name mcentos centos /bin/bash`: 创建一个容器，以bash进行交互
- `docker start [docker-name]`: 启动一个docker
- `doocker stop [docker-name]/[docker-id]`: 停止指定的docker
- `docker exec -it [docker-name] bash`: 指定容器执行bash
- `docker run -id`:  `run`:创建一个新的docker  `-d`: 以后台登陆



- 　--name="nginx-lb":为容器指定一个名称；

  　　-d:后台运行容器，并返回容器ID；

  　　-p:指定映射端口号，本文是将ssh的22端口映射为10022端口，web访问的80端口映射为80端口

  　　-volume: 用来指定挂载目录，将config配置目录、data数据目录、logs日志目录挂载到宿主机上，以后备份方便



```
[root@controller ~]# docker exec -it dockermysql bash
root@3d4b2e0522a0:/# mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.0.18 MySQL Community Server - GPL

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
mysql> GRANT ALL PRIVILEGES ON *.* to 'root'@'%';
Query OK, 0 rows affected (0.05 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)

mysql> alter user 'root'@'%' identified with mysql_native_password by '123456';
Query OK, 0 rows affected (0.01 sec)

mysql> quit
Bye
root@3d4b2e0522a0:/# exit
exit
[root@controller ~]# mysql -h 172.17.0.4 -uroot -p123456
```





### 



docker 创建gitlab仓库

```csharp
docker run \
--detach \
--publish 8443:443 \    # 映射https端口, 不过本文中没有用到
--publish 8090:80 \      # 映射宿主机8090端口到容器中80端口
--publish 8022:22 \      # 映射22端口, 可不配
--name gitlab \            
--restart always \
--hostname 10.12.2.22 \    # 局域网宿主机的ip, 如果是公网主机可以写域名
-v /home/software/gitlab/etc:/etc/gitlab \    # 挂载gitlab的配置文件
-v /home/software/gitlab/logs:/var/log/gitlab \    # 挂载gitlab的日志文件
-v /home/software/gitlab/data:/var/opt/gitlab \    # 挂载gitlab的数据
-v /etc/localtime:/etc/localtime:ro \    # 保持宿主机和容器时间同步
--privileged=true beginor/gitlab-ce    # 在容器中能以root身份执行操作
```