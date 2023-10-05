



# 远程连接adb设备

1. 使用数据线连接手机并进入adb模式
2. 打开adb端口   `adb tcpip 5555`

3. 断开数据线，远程连接   `adb connect 192.168.8.242`





# 根据设备名字连接指定设备

```
adb -s 设备名 shell
```



# 查看安装了的包

```
pm list packages
```





# 安装apk

```
adb install ./xxx.apk
```



# 传输文件

```
adb push xxx.xxx /data/local/tmp/
```



# 端口转发

转发电脑的端口到远程设备的端口

```
adb forward tcp:27042 tcp:27042
adb forward tcp:27043 tcp:27043
```

