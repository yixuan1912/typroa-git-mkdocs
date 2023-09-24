一般通过抓包获取 链接，带data请求等等

如果需要token字段验证或者签名字段，可使用appium加mitmproxy抓包

```python
参考：
https://www.jianshu.com/p/3170dc562997
    Android 7.0以上的手机放弃：mitmproxy无法抓取Android7.0及以上系统的https包，因为系统默认的安全策略已经加固，不支持用户自定义的SSL证书。
需同时满足Appium和mitmproxy两者的要求。（验证adb devices -l，确认android手机已经正常连接，否则需要在手机上打开开发者模式、安装Android SDK等，不在此展开说明了，百度吧。）

```

