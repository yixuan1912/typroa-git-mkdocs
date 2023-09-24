[TOC]


### 1. yum

```BASH
[root@controller yum.repos.d]# yum -y install iptables-services
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
Resolving Dependencies
--> Running transaction check
---> Package iptables-services.x86_64 0:1.4.21-16.el7 will be installed
--> Processing Dependency: iptables = 1.4.21-16.el7 for package: iptables-services-1.4.21-16.el7.x86_64
--> Finished Dependency Resolution
Error: Package: iptables-services-1.4.21-16.el7.x86_64 (centos)
           Requires: iptables = 1.4.21-16.el7
           Installed: iptables-1.4.21-17.el7.x86_64 (@anaconda)
               iptables = 1.4.21-17.el7
           Available: iptables-1.4.21-16.el7.x86_64 (centos)
               iptables = 1.4.21-16.el7
           Installing: iptables-services-1.4.21-16.el7.x86_64 (centos)
               iptables = 1.4.16.1
 You could try using --skip-broken to work around the problem
 You could try running: rpm -Va --nofiles --nodigest

解决：
# yum remove iptables-1.4.21-17.el7.x86_64
# yum  -y install iptables-services
```

### 2. service: command not found

```BASH
# service iptables save
-bash: service: command not found
# yum list | grep initscripts
initscripts.x86_64                      9.49.30-1.el7_2.3              iaas     
# yum install -y initscripts.x86_64
# service iptables save
iptables: Saving firewall rules to /etc/sysconfig/iptables: [  OK  ]
```

### 3. rabbitmq-server

```BASH
# service rabbitmq-server restart
Mar 20 23:22:56 controller rabbitmqctl[6432]: Error: unable to connect to node rabbit@controller: nodedown
Mar 20 23:22:56 controller rabbitmqctl[6432]: DIAGNOSTICS
Mar 20 23:22:56 controller rabbitmqctl[6432]: ===========
Mar 20 23:22:56 controller rabbitmqctl[6432]: attempted to contact: [rabbit@controller]
Mar 20 23:22:56 controller rabbitmqctl[6432]: rabbit@controller:
Mar 20 23:22:56 controller rabbitmqctl[6432]: * unable to connect to epmd (port 4369) on controller: address (cannot connection)

解决：
# 重新更改主机名和hosts文件
```

### 4. mariadb

```BASH
1. 重置密码
# vi /etc/my.cnf
# 在[mysqld]下添加一行skip-grant-tables
#  systemctl restart mariadb
# mysql -u root

MySQL> UPDATE mysql.user SET Password=PASSWORD('新密码') where USER='root';
MySQL> flush privileges;
MySQL> exit
# 把刚才加的skip-grant-tables删除。
#  systemctl restart mariadb
```






