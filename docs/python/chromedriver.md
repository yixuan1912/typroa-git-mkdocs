[TOC]



# (教程)无头浏览器Selenium的使用要点

## 1、无头浏览器介绍

无头浏览器是指可以在图形界面情况下运行的，可以模拟多种浏览器的运行框架。研发可以通过编程来控制该框架执行各种任务，模拟真实的浏览器操作和各种任务，例如登录、js解析、ajax动态生成、获取cookie等。

#### 1.1、无头浏览器适合的场景

无头浏览器的框架需要真实运行浏览器，因此系统开销大，采集运行速度慢，相对与一般的爬虫程序，其运行环境要求搭建的工具和库较多，因此如果目标网站反爬不是很难，可以直接通过简单的http请求进行采集，不适合使用无头浏览器方案。

当目标网站有多种验证机制，例如需要验证登录、ajax动生成、js反爬策略，如果研发不能进行网站行为分析的情况下，建议使用无头浏览器伪装正常用户，同时配合使用爬虫代理加强版进行数据采集。

#### 1.2、无头浏览器框架推荐

无头浏览器方案有很多，我们推荐如下：

- **selenium+chrome+chrome driver**
- **亿牛云爬虫代理加强版**

- **使用mitmproxy进行中继代理**

## 2、安装和使用

#### 2.1、下载chrome对应版本的chrome deriver

下载chrome <https://www.google.com/chrome/>

下载对应版本 driver <https://chromedriver.chromium.org/downloads>

注意chrome版本和driver版本号需要一致，可以查看具体的帮助说明，如果不一致，即使程序能够运行，也会出现爬虫代理认证信息失败，需要弹窗要求手动输入认证信息的问题。

#### 2.2、设置运行模式（防止被网站反爬）

如果浏览器正常运行下，navigator.webdriver的值应该是undefined或者false，如果为true目标网站能检测到selenium，设置为开发者模式，可以防止目标网站识别。

```
from selenium.webdriver import ChromeOptions
option = ChromeOptions()
option.add_experimental_option('excludeSwitches', ['enable-automation'])#开启实验性功能
browser=webdriver.Chrome(options=option)

# 修改get方法
script = '''
Object.defineProperty(navigator, 'webdriver', {
get: () => undefined
})
'''
browser.execute_cdp_cmd("Page.addScriptToEvaluateOnNewDocument", {"source": script})
```

更多反爬资料请参考：

<https://16yun.yuque.com/go/doc/23767204>

<https://16yun.yuque.com/zoxdo5/aorluh/adp4it>

#### 2.3、启动无头浏览器模式

```
# 无头浏览器，需要关闭gpu，启动页面最大化，关闭扩展
option.add_argument("--start-maximized")
option.add_argument('--disable-gpu')
option.add_argument("--headless")  # Runs Chrome in headless mode.
option.add_argument('--no-sandbox')  # Bypass OS security model
option.add_argument('--disable-gpu')  # applicable to windows os only
option.add_argument('start-maximized')  #
option.add_argument('disable-infobars')
option.add_argument("--disable-extensions")
```



#### 2.4、加快页面访问（可能会被反爬）

```
prefs = {
    'profile.default_content_setting_values': {
        'images': 2,  # 屏蔽图片
    }
}
# 添加屏蔽chrome浏览器禁用图片的设置
option.add_experimental_option("prefs", prefs)

# 缓存通用文件
os.makedirs("/tmp/wbtest/custom/profile", exist_ok=True)
option.add_argument("user-data-dir=/tmp/wbtest/custom/profile")

# 过滤掉不需要访问的域名
driver.execute_cdp_cmd(
        'Network.setBlockedURLs', {
            "urls": [
                "m.media-amazon.com",
                "aax-us-east.amazon-adsystem.com",
                "fls-na.amazon.com"
            ]
        }
)
```

#### 2.5、随机user-agent

注意使用的UA需要和浏览器类型匹配，例如：Chrome浏览器请使用Chrome PC的UserAgent

```
ua = random.choice(ua_list)
option.add_argument(
    "user-agent={}".format(ua)
)
```

#### 2.6、每次访问前做清理

```
# 清理所有数据（会导致访问变慢）
driver.get('chrome://settings/clearBrowserData')

# 清理所有cookie
driver.delete_all_cookies()
```

#### 2.7、修改请求过程

需要使用seleniumwire包，详细参考<https://github.com/wkeeling/selenium-wire>

可以通过这个包查看请求过程，帮助完善爬虫程序

```
from seleniumwire import webdriver  # Import from seleniumwire
import json

def interceptor(request):
    # Block PNG, JPEG and GIF images
    if request.path.endswith(('.png', '.jpg', '.gif')):
        request.abort()
        
    if request.url == 'https://server.com/some/path':
        response.headers['New-Header'] = 'Some Value'
        
    if request.method == 'POST' and request.headers['Content-Type'] == 'application/json':
        # The body is in bytes so convert to a string
        body = request.body.decode('utf-8')
        # Load the JSON
        data = json.loads(body)
        # Add a new property
        data['foo'] = 'bar'
        # Set the JSON back on the request
        request.body = json.dumps(data).encode('utf-8')
        # Update the content length
        del request.headers['Content-Length']
        request.headers['Content-Length'] = str(len(request.body))    
        
# Create a new instance of the Chrome driver
driver = webdriver.Chrome()

driver.request_interceptor = interceptor

# Go to the Google home page
driver.get('https://www.baidu.com')

# Access requests via the `requests` attribute
for request in driver.requests:
    if request.response:
        print(
            request.url,
            request.response.status_code,
            request.response.headers
        )
```

#### 

## 3、配合使用爬虫代理加强版

通过无头浏览器模拟用户操作，同时结合爬虫代理加强版实现IP地址自动切换，可以真实的实现用户终端请求，获取相应的数据，下面是获取cookie的代码：

```
    import os
    import time
    import zipfile

    from selenium import webdriver
    from selenium.common.exceptions import TimeoutException
    from selenium.webdriver.common.by import By
    from selenium.webdriver.support import expected_conditions as EC
    from selenium.webdriver.support.ui import WebDriverWait


    class GenCookies(object):
        # 随机useragent
        USER_AGENT = open('useragents.txt').readlines()


        # 代理服务器(产品官网 www.16yun.cn)
        PROXY_HOST = 't.16yun.cn'  #  proxy or host
        PROXY_PORT = 31111  # port
        PROXY_USER = 'USERNAME'  # username
        PROXY_PASS = 'PASSWORD'  # password

        @classmethod
        def get_chromedriver(cls, use_proxy=False, user_agent=None):
            manifest_json = """
            {
                "version": "1.0.0",
                "manifest_version": 2,
                "name": "Chrome Proxy",
                "permissions": [
                    "proxy",
                    "tabs",
                    "unlimitedStorage",
                    "storage",
                    "<all_urls>",
                    "webRequest",
                    "webRequestBlocking"
                ],
                "background": {
                    "scripts": ["background.js"]
                },
                "minimum_chrome_version":"22.0.0"
            }
            """

            background_js = """
            var config = {
                    mode: "fixed_servers",
                    rules: {
                    singleProxy: {
                        scheme: "http",
                        host: "%s",
                        port: parseInt(%s)
                    },
                    bypassList: ["localhost"]
                    }
                };

            chrome.proxy.settings.set({value: config, scope: "regular"}, function() {});

            function callbackFn(details) {
                return {
                    authCredentials: {
                        username: "%s",
                        password: "%s"
                    }
                };
            }

            chrome.webRequest.onAuthRequired.addListener(
                        callbackFn,
                        {urls: ["<all_urls>"]},
                        ['blocking']
            );
            """ % (cls.PROXY_HOST, cls.PROXY_PORT, cls.PROXY_USER, cls.PROXY_PASS)
            path = os.path.dirname(os.path.abspath(__file__))
            chrome_options = webdriver.ChromeOptions()

            # 关闭webdriver的一些标志
            # chrome_options.add_experimental_option('excludeSwitches', ['enable-automation'])        


            if use_proxy:
                pluginfile = 'proxy_auth_plugin.zip'

                with zipfile.ZipFile(pluginfile, 'w') as zp:
                    zp.writestr("manifest.json", manifest_json)
                    zp.writestr("background.js", background_js)
                chrome_options.add_extension(pluginfile)
            if user_agent:
                chrome_options.add_argument('--user-agent=%s' % user_agent)
            driver = webdriver.Chrome(
                os.path.join(path, 'chromedriver'),
                chrome_options=chrome_options)

            # 修改webdriver get属性
            # script = '''
            # Object.defineProperty(navigator, 'webdriver', {
            # get: () => undefined
            # })
            # '''
            # driver.execute_cdp_cmd("Page.addScriptToEvaluateOnNewDocument", {"source": script})

            return driver

        def __init__(self, username, password):        
            # 登录example网站
            self.url = 'https://passport.test.cn/signin/login?entry=example&r=https://m.test.cn/'
            self.browser = self.get_chromedriver(use_proxy=True, user_agent=self.USER_AGENT)
            self.wait = WebDriverWait(self.browser, 20)
            self.username = username
            self.password = password

        def open(self):
            """
            打开网页输入用户名密码并点击
            :return: None
            """
            self.browser.delete_all_cookies()
            self.browser.get(self.url)
            username = self.wait.until(EC.presence_of_element_located((By.ID, 'loginName')))
            password = self.wait.until(EC.presence_of_element_located((By.ID, 'loginPassword')))
            submit = self.wait.until(EC.element_to_be_clickable((By.ID, 'loginAction')))
            username.send_keys(self.username)
            password.send_keys(self.password)
            time.sleep(1)
            submit.click()

        def password_error(self):
            """
            判断是否密码错误
            :return:
            """
            try:
                return WebDriverWait(self.browser, 5).until(
                    EC.text_to_be_present_in_element((By.ID, 'errorMsg'), '用户名或密码错误'))
            except TimeoutException:
                return False

        def get_cookies(self):
            """
            获取Cookies
            :return:
            """
            return self.browser.get_cookies()

        def main(self):
            """
            入口
            :return:
            """
            self.open()
            if self.password_error():
                return {
                    'status': 2,
                    'content': '用户名或密码错误'
                }            

            cookies = self.get_cookies()
            return {
                'status': 1,
                'content': cookies
            }


    if __name__ == '__main__':
        result = GenCookies(
            username='180000000',
            password='16yun',
        ).main()
        print(result)
        
```





## 4、配合使用API优质代理

```python
import requests
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
import os


def get_proxy_list(url, timeout=10):
    try:
        resp = requests.get(url, timeout=timeout)
        print(resp.text)
        if resp.status_code != 200:
            print(("http status code {}".format(resp.status_code)))
            return []
        data = [l.strip() for l in resp.text.split() if len(l.strip()) > 0]
        return data
    except Exception as err:
        print(err)
        return []
    
def get_data(ua, proxy, url):
    opts = Options()
    opts.add_experimental_option('excludeSwitches', ['enable-automation'])

    opts.add_argument('--proxy-server=%s' % proxy)
    opts.add_argument(
        "user-agent={}".format(ua)
    )
    prefs = {
        'profile.default_content_setting_values': {
            'images': 2,  # 屏蔽图片
        }
    }
    # 添加屏蔽chrome浏览器禁用图片的设置
    opts.add_experimental_option("prefs", prefs)


    # 缓存通用文件
    os.makedirs("/tmp/wbtest/custom/profile", exist_ok=True)
    opts.add_argument("user-data-dir=/tmp/wbtest/custom/profile")

    # 关闭显示
    opts.add_argument("--start-maximized")
    opts.add_argument('--disable-gpu')
    opts.add_argument("--headless")  # Runs Chrome in headless mode.
    opts.add_argument('--no-sandbox')  # Bypass OS security model
    opts.add_argument('--disable-gpu')  # applicable to windows os only
    opts.add_argument('start-maximized')  #
    opts.add_argument('disable-infobars')
    opts.add_argument("--disable-extensions")


    # 修改webdriver get属性
    script = '''
    Object.defineProperty(navigator, 'webdriver', {
    get: () => undefined
    })
    '''

    dr = webdriver.Chrome(executable_path="./chromdriver", options=opts)
    dr.execute_cdp_cmd("Page.addScriptToEvaluateOnNewDocument", {"source": script})

    # 清理所有数据
    # dr.get('chrome://settings/clearBrowserData')
    
    # 清理cookie
    dr.delete_all_cookies()


    dr.get(url)

    cookie = dr.get_cookies()
    html = dr.page_source
    dr.close()
    dr.quit()

    return cookie, html

if __name__ == '__main__':    
    import random

    proxylist = get_proxy_list(
        "http://ip.16yun.cn/myip/pl/xxxxxxxx"
    )


    for i in range(10):
        proxy = random.choice(proxylist)
        ua = "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.131 Safari/537.36"

        cookie, html = get_data(ua=ua, proxy=proxy, url="https://www.baidu.com")        
        print(html)
```





# 无头浏览器反爬-Linux虚拟屏幕Xvfb

Xvfb是流行的虚拟现实库，可以使很多需要图形界面的程序虚拟运行；



使用原因
淘宝列表页爬取经常会跳滑动验证码，需要使用puppeteer控制浏览器模拟人工滑动，而在无头模式下，无论怎么滑动都无法通过，必须使用有头模式，而CentOS服务器上没有界面，所以只能使用虚拟屏幕，使浏览器运行在虚拟屏幕上。



可以使用docker版本的Xvfb镜像 <https://github.com/joyzoursky/docker-python-chromedriver>



安装及使用



```
# 安装
yum install Xvfb

# 启动  7:虚拟屏幕id，使用时需要用到；  1336x768x24:屏幕分辨率；
Xvfb :7 -screen 0 1336x768x24 2>/dev/null &

# 使用    启动puppeteer程序前需指定运行在哪个虚拟屏幕上，如下命令，运行在虚拟屏幕id为7的上
export DISPLAY=:7
```

##  远程控制、监控 

一般情况下，Xvfb都运行在Linux服务器上，你无法看到虚拟屏幕的运行情况，程序的调试十分麻烦，所以你需要在本地远程连接你服务器上所在的虚拟屏幕，进行监控控制。



服务器端



x11vnc：远程桌面，可以使人远程连接服务器上的虚拟屏幕；



```
# 安装 ubuntu
sudo apt install x11vnc

# 安装 centos
yum install -y x11vnc

# 启动  5900:连接端口  password:连接密码  7:虚拟屏幕id
x11vnc -listen 0.0.0.0 -rfbport 5900 -noipv6 -passwd password -display :7
```

##  用户端



可以使用vncviewer工具远程连接。



其它
python有一个第三方库PyVirtualDisplay，该库可以在python程序中启动指定运行端口xvfb程序，并且可远程监听。











# 无头浏览器反爬

## 无头浏览器（**chrome headless** 的检测以及反检测）。

无头浏览器的检测应该是爬虫中非常重要的一块，基础资料可以参考 [not-possible-to-block-chrome-headless](https://intoli.com/blog/not-possible-to-block-chrome-headless/) 这个文章，大概是最初级的无头浏览器检测方案。不过在后续的爬虫中，有一种魔高一尺，道高一丈的感觉，陆续发现很多站点都能通过各种其他手法检测无头浏览器。

再后来，看一些网站的js，发现了他们会对很多字段进行收集，都与[fingerprintjs2](https://github.com/Valve/fingerprintjs2) 收集的差不多，才算是豁然开朗。

故在此记录一下可以被用于检测无头浏览器的字段即如果绕过去。

- userAgent
  ua可以说是最基础的字段了，然是无头浏览器中，单纯修改ua还不行，需要ua 和一些字段（如navigator.platform）跟着修改才行。当然，单纯的ua 还是要尽量模仿桌面浏览器的
  识别：通过 navigator.userAgent 通过ua来识别是否为爬虫
  反识别：可以修改属性来绕过

```
Object.defineProperty(navigator, 'userAgent', {
    get: () => 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/74.0.3729.169 Safari/537.36'
});
```

- webdriver

The **webdriver** read-only property of the `navigator` interface indicates whether the user agent is controlled by automation.

- 识别：桌面chrome 浏览器navigator.webdriver 返回为undefined，故可以判断`!!navigator.webdriver`
  反识别：

```
Object.defineProperty(navigator, 'webdriver', {
    get: () => false
});
```

- 或者:

```
delete navigator.__proto__.webdriver
```

- 发现过有网站监测`navigator.webdriver` 是否为undefined，而不是简单的通过`!!navigator.webdriver`进行监测的
- language
  不同浏览器对language 支持不同，chrome只有navigator.language  [参考](http://www.w3help.org/zh-cn/causes/BX2040)
  识别： navigator.language || navigator.userLanguage || navigator.browserLanguage || navigator.systemLanguage 是否存在
  反识别：

```
Object.defineProperty(navigator, 'language', {
	get: () => "zh-CN",
});
```

- colorDepth
  window.screen.colorDepth 屏幕颜色深度，一般收集指纹的时候会用到。无头浏览器这个字段一般与桌面浏览器相同
- deviceMemory
  只读属性返回千兆字节为单位的大概的机器内存。这个值是一个2的次方数除以1024，舍去小数点的近似值。不过这还只是个实验特性，不一定所有浏览器都有。指纹收集的时候会使用到，以防万一可以设个值
  navigator.deviceMemory
  反识别:

```
Object.defineProperty(navigator, 'deviceMemory', {
	get: () => 8
});
```

- hardwareConcurrency
  浏览器环境所拥有的CPU核心数
  navigator.hardwareConcurrency
  反识别:

```
Object.defineProperty(navigator, 'hardwareConcurrency', {
	get: () => 8
});
```

- screenResolution
  会被用于指纹
  [window.screen.width, window.screen.height]
- availableScreenResolution
  会被用于指纹
  [window.screen.availHeight, window.screen.availWidth]

- timezoneOffset
  调世界时（UTC）相对于当前时区的时间差值，单位为分钟。
  new Date().getTimezoneOffset()
- timezone
  new window.Intl.DateTimeFormat().resolvedOptions().timeZone

- sessionStorage *
  只判断是否有sessionStorage
  !!window.sessionStorage
- localStorage *
  !!window.localStorage

- indexedDb
  !!window.indexedDB
- addBehavior
  该字段为IE特有 [参考](http://www.w3help.org/zh-cn/causes/BT9027)
  !!(document.body && document.body.addBehavior)

- openDatabase *
  会返回一个databse，可以执行sql。 指纹只判断是否有这个方法
  !!window.openDatabase
- cpuClass
  cpu 类型，似乎chrome 浏览器返回为undefined
  navigator.cpuClass

- platform *
  navigator.platform
  识别： if (navigator.platform === Linux x86_64) Linux x86_64占有率较小，如果为Linux x86_64，大概率为爬虫
  反识别:

```
Object.defineProperty(navigator, 'platform', {
	get: () => 'MacIntel'
});
```

- plugins
  navigator.plugins
  plugin对象有name filename description version 属性。可以使用plugin[0] 的方式获取MimeType对象
- mimeTypes
  navigator.mimeTypes
  mimeTypes对象需要和plugin 对象对应，有type, suffixes, description, enabledPlugin 属性

- canvas
  获取canvas 指纹  [参考](https://browserleaks.com/canvas)， 通过指纹来判别一个浏览器，并根据指纹的一些特征识别是何种浏览器。
  识别:

```
(function (name, context, definition) {
        if (typeof module !== 'undefined' && module.exports) {
            module.exports = definition();
        }
        else if (typeof define === 'function' && define.amd) {
            define(definition);
        }
        else {
            context[name] = definition();
        }
    })('Fingerprint', this, function () {
        'use strict';
  
        var Fingerprint = function (options) {
            var nativeForEach, nativeMap;
            nativeForEach = Array.prototype.forEach;
            nativeMap = Array.prototype.map;
  
            this.each = function (obj, iterator, context) {
                if (obj === null) {
                    return;
                }
                if (nativeForEach && obj.forEach === nativeForEach) {
                    obj.forEach(iterator, context);
                } else if (obj.length === +obj.length) {
                    for (var i = 0, l = obj.length; i < l; i++) {
                        if (iterator.call(context, obj[i], i, obj) === {}) return;
                    }
                } else {
                    for (var key in obj) {
                        if (obj.hasOwnProperty(key)) {
                            if (iterator.call(context, obj[key], key, obj) === {}) return;
                        }
                    }
                }
            };
  
            this.map = function (obj, iterator, context) {
                var results = [];
                // Not using strict equality so that this acts as a
                // shortcut to checking for `null` and `undefined`.
                if (obj == null) return results;
                if (nativeMap && obj.map === nativeMap) return obj.map(iterator, context);
                this.each(obj, function (value, index, list) {
                    results[results.length] = iterator.call(context, value, index, list);
                });
                return results;
            };
  
            if (typeof options == 'object') {
                this.hasher = options.hasher;
                this.screen_resolution = options.screen_resolution;
                this.screen_orientation = options.screen_orientation;
                this.canvas = options.canvas;
                this.ie_activex = options.ie_activex;
            } else if (typeof options == 'function') {
                this.hasher = options;
            }
        };
  
        Fingerprint.prototype = {
            get: function () {
                var keys = [];
                keys.push(navigator.userAgent);
                keys.push(navigator.language);
                keys.push(screen.colorDepth);
                if (this.screen_resolution) {
                    var resolution = this.getScreenResolution();
                    if (typeof resolution !== 'undefined') { // headless browsers, such as phantomjs
                        keys.push(resolution.join('x'));
                    }
                }
                keys.push(new Date().getTimezoneOffset());
                keys.push(this.hasSessionStorage());
                keys.push(this.hasLocalStorage());
                keys.push(this.hasIndexDb());
                //body might not be defined at this point or removed programmatically
                if (document.body) {
                    keys.push(typeof(document.body.addBehavior));
                } else {
                    keys.push(typeof undefined);
                }
                keys.push(typeof(window.openDatabase));
                keys.push(navigator.cpuClass);
                keys.push(navigator.platform);
                keys.push(navigator.doNotTrack);
                keys.push(this.getPluginsString());
                if (this.canvas && this.isCanvasSupported()) {
                    keys.push(this.getCanvasFingerprint());
                }
                if (this.hasher) {
                    return this.hasher(keys.join('###'), 31);
                } else {
                    return this.murmurhash3_32_gc(keys.join('###'), 31);
                }
            },
  
            /**
             * JS Implementation of MurmurHash3 (r136) (as of May 20, 2011)
             *
             * @author <a href="mailto:gary.court@gmail.com">Gary Court</a>
             * @see http://github.com/garycourt/murmurhash-js
             * @author <a href="mailto:aappleby@gmail.com">Austin Appleby</a>
             * @see http://sites.google.com/site/murmurhash/
             *
             * @param {string} key ASCII only
             * @param {number} seed Positive integer only
             * @return {number} 32-bit positive integer hash
             */
  
            murmurhash3_32_gc: function (key, seed) {
                var remainder, bytes, h1, h1b, c1, c2, k1, i;
  
                remainder = key.length & 3; // key.length % 4
                bytes = key.length - remainder;
                h1 = seed;
                c1 = 0xcc9e2d51;
                c2 = 0x1b873593;
                i = 0;
  
                while (i < bytes) {
                    k1 =
                        ((key.charCodeAt(i) & 0xff)) |
                        ((key.charCodeAt(++i) & 0xff) << 8) |
                        ((key.charCodeAt(++i) & 0xff) << 16) |
                        ((key.charCodeAt(++i) & 0xff) << 24);
                    ++i;
  
                    k1 = ((((k1 & 0xffff) * c1) + ((((k1 >>> 16) * c1) & 0xffff) << 16))) & 0xffffffff;
                    k1 = (k1 << 15) | (k1 >>> 17);
                    k1 = ((((k1 & 0xffff) * c2) + ((((k1 >>> 16) * c2) & 0xffff) << 16))) & 0xffffffff;
  
                    h1 ^= k1;
                    h1 = (h1 << 13) | (h1 >>> 19);
                    h1b = ((((h1 & 0xffff) * 5) + ((((h1 >>> 16) * 5) & 0xffff) << 16))) & 0xffffffff;
                    h1 = (((h1b & 0xffff) + 0x6b64) + ((((h1b >>> 16) + 0xe654) & 0xffff) << 16));
                }
  
                k1 = 0;
  
                switch (remainder) {
                    case 3:
                        k1 ^= (key.charCodeAt(i + 2) & 0xff) << 16;
                    case 2:
                        k1 ^= (key.charCodeAt(i + 1) & 0xff) << 8;
                    case 1:
                        k1 ^= (key.charCodeAt(i) & 0xff);
  
                        k1 = (((k1 & 0xffff) * c1) + ((((k1 >>> 16) * c1) & 0xffff) << 16)) & 0xffffffff;
                        k1 = (k1 << 15) | (k1 >>> 17);
                        k1 = (((k1 & 0xffff) * c2) + ((((k1 >>> 16) * c2) & 0xffff) << 16)) & 0xffffffff;
                        h1 ^= k1;
                }
  
                h1 ^= key.length;
  
                h1 ^= h1 >>> 16;
                h1 = (((h1 & 0xffff) * 0x85ebca6b) + ((((h1 >>> 16) * 0x85ebca6b) & 0xffff) << 16)) & 0xffffffff;
                h1 ^= h1 >>> 13;
                h1 = ((((h1 & 0xffff) * 0xc2b2ae35) + ((((h1 >>> 16) * 0xc2b2ae35) & 0xffff) << 16))) & 0xffffffff;
                h1 ^= h1 >>> 16;
  
                return h1 >>> 0;
            },
  
            // https://bugzilla.mozilla.org/show_bug.cgi?id=781447
            hasLocalStorage: function () {
                try {
                    return !!window.localStorage;
                } catch (e) {
                    return true; // SecurityError when referencing it means it exists
                }
            },
  
            hasSessionStorage: function () {
                try {
                    return !!window.sessionStorage;
                } catch (e) {
                    return true; // SecurityError when referencing it means it exists
                }
            },
  
            hasIndexDb: function () {
                try {
                    return !!window.indexedDB;
                } catch (e) {
                    return true; // SecurityError when referencing it means it exists
                }
            },
  
            isCanvasSupported: function () {
                var elem = document.createElement('canvas');
                return !!(elem.getContext && elem.getContext('2d'));
            },
  
            isIE: function () {
                if (navigator.appName === 'Microsoft Internet Explorer') {
                    return true;
                } else if (navigator.appName === 'Netscape' && /Trident/.test(navigator.userAgent)) {// IE 11
                    return true;
                }
                return false;
            },
  
            getPluginsString: function () {
                if (this.isIE() && this.ie_activex) {
                    return this.getIEPluginsString();
                } else {
                    return this.getRegularPluginsString();
                }
            },
  
            getRegularPluginsString: function () {
                return this.map(navigator.plugins, function (p) {
                    var mimeTypes = this.map(p, function (mt) {
                        return [mt.type, mt.suffixes].join('~');
                    }).join(',');
                    return [p.name, p.description, mimeTypes].join('::');
                }, this).join(';');
            },
  
            getIEPluginsString: function () {
                if (window.ActiveXObject) {
                    var names = ['ShockwaveFlash.ShockwaveFlash',//flash plugin
                        'AcroPDF.PDF', // Adobe PDF reader 7+
                        'PDF.PdfCtrl', // Adobe PDF reader 6 and earlier, brrr
                        'QuickTime.QuickTime', // QuickTime
                        // 5 versions of real players
                        'rmocx.RealPlayer G2 Control',
                        'rmocx.RealPlayer G2 Control.1',
                        'RealPlayer.RealPlayer(tm) ActiveX Control (32-bit)',
                        'RealVideo.RealVideo(tm) ActiveX Control (32-bit)',
                        'RealPlayer',
                        'SWCtl.SWCtl', // ShockWave player
                        'WMPlayer.OCX', // Windows media player
                        'AgControl.AgControl', // Silverlight
                        'Skype.Detection'];
  
                    // starting to detect plugins in IE
                    return this.map(names, function (name) {
                        try {
                            new ActiveXObject(name);
                            return name;
                        } catch (e) {
                            return null;
                        }
                    }).join(';');
                } else {
                    return ""; // behavior prior version 0.5.0, not breaking backwards compat.
                }
            },
  
            getScreenResolution: function () {
                var resolution;
                if (this.screen_orientation) {
                    resolution = (screen.height > screen.width) ? [screen.height, screen.width] : [screen.width, screen.height];
                } else {
                    resolution = [screen.height, screen.width];
                }
                return resolution;
            },
  
            getCanvasFingerprint: function () {
                var canvas = document.createElement('canvas');
                var ctx = canvas.getContext('2d');
                // https://www.browserleaks.com/canvas#how-does-it-work
                var txt = 'http://valve.github.io';
                ctx.textBaseline = "top";
                ctx.font = "14px 'Arial'";
                ctx.textBaseline = "alphabetic";
                ctx.fillStyle = "#f60";
                ctx.fillRect(125, 1, 62, 20);
                ctx.fillStyle = "#069";
                ctx.fillText(txt, 2, 15);
                ctx.fillStyle = "rgba(102, 204, 0, 0.7)";
                ctx.fillText(txt, 4, 17);
                return canvas.toDataURL();
            }
        };
        return Fingerprint;
    });
  
    new Fingerprint({canvas: true}).get();
```

- 反识别: 通过重写toBlob 以及toDataURL 方法来解决

```
var inject = function () {
    var overwrite = function (name) {
      const OLD = HTMLCanvasElement.prototype[name];
      Object.defineProperty(HTMLCanvasElement.prototype, name, {
        "value": function () {
          var shift = {
            'r': Math.floor(Math.random() * 10) - 5,
            'g': Math.floor(Math.random() * 10) - 5,
            'b': Math.floor(Math.random() * 10) - 5,
            'a': Math.floor(Math.random() * 10) - 5
          };
          var width = this.width, height = this.height, context = this.getContext("2d");
          var imageData = context.getImageData(0, 0, width, height);
          for (var i = 0; i < height; i++) {
            for (var j = 0; j < width; j++) {
              var n = ((i * (width * 4)) + (j * 4));
              imageData.data[n + 0] = imageData.data[n + 0] + shift.r;
              imageData.data[n + 1] = imageData.data[n + 1] + shift.g;
              imageData.data[n + 2] = imageData.data[n + 2] + shift.b;
              imageData.data[n + 3] = imageData.data[n + 3] + shift.a;
            }
          }
          context.putImageData(imageData, 0, 0);
          return OLD.apply(this, arguments);
        }
      });
    };
    overwrite('toBlob');
    overwrite('toDataURL');
  };
  inject();
```

- webgl
  TODO
- webglVendorAndRenderer
  返会显卡型号相关信息
  TODO

- adBlock
  检测adBlock 插件是否存在
- hasLiedLanguages
  判断navigator.language 是否为navigator.languages数组的第一个

- hasLiedResolution
  window.screen.width < window.screen.availWidth || window.screen.height < window.screen.availHeight
- hasLiedOs
  识别: 判断ua 和platform 之间的关联。

```
var getHasLiedOs = function () {
    var userAgent = navigator.userAgent.toLowerCase()
    var oscpu = navigator.oscpu
    var platform = navigator.platform.toLowerCase()
    var os
    // We extract the OS from the user agent (respect the order of the if else if statement)
    if (userAgent.indexOf('windows phone') >= 0) {
      os = 'Windows Phone'
    } else if (userAgent.indexOf('win') >= 0) {
      os = 'Windows'
    } else if (userAgent.indexOf('android') >= 0) {
      os = 'Android'
    } else if (userAgent.indexOf('linux') >= 0 || userAgent.indexOf('cros') >= 0) {
      os = 'Linux'
    } else if (userAgent.indexOf('iphone') >= 0 || userAgent.indexOf('ipad') >= 0) {
      os = 'iOS'
    } else if (userAgent.indexOf('mac') >= 0) {
      os = 'Mac'
    } else {
      os = 'Other'
    }
    // We detect if the person uses a mobile device
    var mobileDevice = (('ontouchstart' in window) ||
      (navigator.maxTouchPoints > 0) ||
      (navigator.msMaxTouchPoints > 0))
  
    if (mobileDevice && os !== 'Windows Phone' && os !== 'Android' && os !== 'iOS' && os !== 'Other') {
      return true
    }
  
    console.log(oscpu);
    // We compare oscpu with the OS extracted from the UA
    if (typeof oscpu !== 'undefined') {
      oscpu = oscpu.toLowerCase()
      if (oscpu.indexOf('win') >= 0 && os !== 'Windows' && os !== 'Windows Phone') {
        return true
      } else if (oscpu.indexOf('linux') >= 0 && os !== 'Linux' && os !== 'Android') {
        return true
      } else if (oscpu.indexOf('mac') >= 0 && os !== 'Mac' && os !== 'iOS') {
        return true
      } else if ((oscpu.indexOf('win') === -1 && oscpu.indexOf('linux') === -1 && oscpu.indexOf('mac') === -1) !== (os === 'Other')) {
        return true
      }
    }
  
    // We compare platform with the OS extracted from the UA
    if (platform.indexOf('win') >= 0 && os !== 'Windows' && os !== 'Windows Phone') {
      return true
    } else if ((platform.indexOf('linux') >= 0 || platform.indexOf('android') >= 0 || platform.indexOf('pike') >= 0) && os !== 'Linux' && os !== 'Android') {
      return true
    } else if ((platform.indexOf('mac') >= 0 || platform.indexOf('ipad') >= 0 || platform.indexOf('ipod') >= 0 || platform.indexOf('iphone') >= 0) && os !== 'Mac' && os !== 'iOS') {
      return true
    } else {
      var platformIsOther = platform.indexOf('win') < 0 &&
        platform.indexOf('linux') < 0 &&
        platform.indexOf('mac') < 0 &&
        platform.indexOf('iphone') < 0 &&
        platform.indexOf('ipad') < 0
      if (platformIsOther !== (os === 'Other')) {
        return true
      }
    }
  
    return typeof navigator.plugins === 'undefined' && os !== 'Windows' && os !== 'Windows Phone'
  }
```

- 反识别： 确保ua 以及platform 之间的关系
- hasLiedBrowser
  识别: 根据ua 判断browser，在根据一些神奇的特性去判断。。

```
var getHasLiedBrowser = function () {
    var userAgent = navigator.userAgent.toLowerCase()
    var productSub = navigator.productSub
  
    // we extract the browser from the user agent (respect the order of the tests)
    var browser
    if (userAgent.indexOf('firefox') >= 0) {
      browser = 'Firefox'
    } else if (userAgent.indexOf('opera') >= 0 || userAgent.indexOf('opr') >= 0) {
      browser = 'Opera'
    } else if (userAgent.indexOf('chrome') >= 0) {
      browser = 'Chrome'
    } else if (userAgent.indexOf('safari') >= 0) {
      browser = 'Safari'
    } else if (userAgent.indexOf('trident') >= 0) {
      browser = 'Internet Explorer'
    } else {
      browser = 'Other'
    }
  
    if ((browser === 'Chrome' || browser === 'Safari' || browser === 'Opera') && productSub !== '20030107') {
      return true
    }
  
    // eslint-disable-next-line no-eval
    var tempRes = eval.toString().length
    if (tempRes === 37 && browser !== 'Safari' && browser !== 'Firefox' && browser !== 'Other') {
      return true
    } else if (tempRes === 39 && browser !== 'Internet Explorer' && browser !== 'Other') {
      return true
    } else if (tempRes === 33 && browser !== 'Chrome' && browser !== 'Opera' && browser !== 'Other') {
      return true
    }
  
    // We create an error to see how it is handled
    var errFirefox
    try {
      // eslint-disable-next-line no-throw-literal
      throw 'a'
    } catch (err) {
      try {
        err.toSource()
        errFirefox = true
      } catch (errOfErr) {
        errFirefox = false
      }
    }
    return errFirefox && browser !== 'Firefox' && browser !== 'Other'
  }
```

- touchSupport

```
var getTouchSupport = function () {
    var maxTouchPoints = 0
    var touchEvent
    if (typeof navigator.maxTouchPoints !== 'undefined') {
      maxTouchPoints = navigator.maxTouchPoints
    } else if (typeof navigator.msMaxTouchPoints !== 'undefined') {
      maxTouchPoints = navigator.msMaxTouchPoints
    }
    try {
      document.createEvent('TouchEvent')
      touchEvent = true
    } catch (_) {
      touchEvent = false
    }
    var touchStart = 'ontouchstart' in window
    // [0, false, false]
    return [maxTouchPoints, touchEvent, touchStart]
  }
```

- fonts
  看有多少个font
- audio
  音频指纹 TODO





# 无头浏览器(PHP-WebDriver-Chrome)

[Php-webdriver](https://github.com/php-webdriver/php-webdriver) 是 Facebook 开发的基于 PHP 语言实现的 Selenium WebDriver 客户端组件，可以用它来操作浏览器。常见的操作包括：自动化测试、采集数据等。

### 安装浏览器（Google Chrome ）Firefox新版本支持有问题，不建议使用

**以 Ubuntu server 16.04 安装 Google Chrome 浏览器为例（**[**参考链接**](https://askubuntu.com/a/760452/470148)**）。**

下载最新版的 64 位 Google Chrome：

```
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
```

安装 Google Chrome 并自动安装依赖：

```
sudo dpkg -i --force-depends google-chrome-stable_current_amd64.deb
```

若提示 `dependency problems` 出错信息，表示某些依赖库没有安装，键入以下命令来自动安装好依赖：

```
sudo apt-get install -f
```

测试 Google Chrome：

```
# 安装中文字体
sudo apt-get install fonts-noto-cjk
# 运行 Google Chrome headless 并截图保存
LANGUAGE=ZH-CN.UTF-8 google-chrome --headless https://www.pc6.com --no-sandbox --screenshot --window-size=1400,900
```

看到如下的图片就表明安装成功了。

![img](https://cdn.nlark.com/yuque/0/2021/png/1313084/1610504205682-4dcba933-30bb-42af-9f73-57145c853bfd.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_40%2Ctext_5Lq_54mb5LqRIHd3dy4xNnl1bi5jbg%3D%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

### 安装浏览器驱动程序（Chromedriver 或 Geckodriver）

**以安装 Chromdriver 为例。**

要保证 Chromedriver 和 Google Chrome 是相匹配的版本。在 Chromedriver 的[官方下载页面](https://chromedriver.chromium.org/downloads)有版本说明，按照需要下载。

如果安装的 Google Chrome 的版本号是 81，那么根据 Chromdriver 官方说明，也需要下载对应的版本号是 81 的 Chromdriver。

```
# 查询 Google Chrome 的版本号
root@aeb9f39e9e04:/tmp# google-chrome --version
Google Chrome 81.0.4044.92
```

![img](https://cdn.nlark.com/yuque/0/2021/png/1313084/1610504205565-dcf00355-ac07-45a1-9a31-c52f6d070d9b.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_18%2Ctext_5Lq_54mb5LqRIHd3dy4xNnl1bi5jbg%3D%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

下载完成后，解压缩出二进制的可执行文件，这就是我们需要的浏览器驱动程序（WebDriver）了。通过 Chromedriver 可以控制 Google Chrome 的操作。

启动 Chromedriver 并监听 4444 端口。

```
LANGUAGE=ZH-CN.UTF-8 ./chromedriver --port=4444
```

### 安装 Php-webdriver

当做完前面两步准备工作，安装好了浏览器（Google Chrome）与浏览器驱动程序（Chromdriver）之后，总算可以进入主题，安装与使用 Php-webdriver 了。

**安装 Php-webdriver**

```
composer require php-webdriver/webdriver
```

#### 配置爬虫代理

```
$pluginForProxyLogin = '/tmp/a'.uniqid().'.zip';

$zip = new ZipArchive();
$res = $zip->open($pluginForProxyLogin, ZipArchive::CREATE | ZipArchive::OVERWRITE);
$zip->addFile('/path/to/Chrome-proxy-helper/manifest.json', 'manifest.json');
$background = file_get_contents('/path/to/Chrome-proxy-helper/background.js');
$background = str_replace(['%proxy_host', '%proxy_port', '%username', '%password'], ['t.16yun.cn', '31111', 'username', 'password'], $background);
$zip->addFromString('background.js', $background);
$zip->close();

putenv("webdriver.chrome.driver=/path/to/chromedriver");

$options = new ChromeOptions();
$options->addExtensions([$pluginForProxyLogin]);
$caps = DesiredCapabilities::chrome();
$caps->setCapability(ChromeOptions::CAPABILITY, $options);

$driver = ChromeDriver::start($caps);
$driver->get('https://httpbin.org/ip');

header('Content-Type: image/png');
echo $driver->takeScreenshot();


unlink($pluginForProxyLogin);
```



### 使用 Php-webdriver

#### 打开浏览器

```
$options = new ChromeOptions();
$options->addArguments([
     '--window-size=1400,900',
     '--headless',
]);
$capabilities = DesiredCapabilities::chrome();
$capabilities->setCapability(ChromeOptions::CAPABILITY, $options);
$host = 'http://localhost:4444';
$driver = RemoteWebDriver::create($host, $capabilities);
```

以上代码加上了打开浏览器的同时加上了窗口大小和无头浏览器的参数，可以按需增减。更多关于 ChromeOptions 的参数请查看 <https://sites.google.com/a/chromium.org/chromedriver/capabilities>。

#### 打开 URL

```
$driver->get('https://www.baidu.com/');
$driver->navigate()->to('https://www.sogou.com/');
```

#### 刷新页面

```
$driver->navigate()->refresh();
```

#### 查找单个元素

```
$element = $driver->findElement(WebDriverBy::cssSelector('div.header'));
```

#### 查找多个元素

```
$elements = $driver->findElements(WebDriverBy::cssSelector('ul.foo > li'));
```

WebDriverBy 提供了多种查询方式：

`WebDriverBy::id($id)` 根据 ID 查找元素

`WebDriverBy::className($className)` 根据 class 查找元素

`WebDriverBy::cssSelector($selctor)` 根据通用的 css 选择器查询

`WebDriverBy::name($name)` 根据元素的 name 属性查询

`WebDriverBy::linkText($text)` 根据可见元素的文本锚点查询

`WebDriverBy::tagName($tagName)` 根据元素标签名称查询

`WebDriverBy::xpath($xpath)` 根据 xpath 表达式查询

#### 输入内容

```
$driver->findElement(WebDriverBy::cssSelector('input[name=username]'))->sendKeys('imzhi');
```

#### 点击元素

```
$driver->findElement(WebDriverBy::cssSelector('button#login'))->click();
```

#### 截图

```
$driver->takeScreenshot(__DIR__ . '/screenshot.png');
```

#### 执行 JS

`RemoteWebDriver::executeScript($script, $args)` 执行同步 JS 代码

`RemoteWebDriver::executeAsyncScript($script, $args)` 执行异步 JS 代码

```
$driver->executeScript("document.body.style.backgroundColor = 'red';");
```

关于 Php-webdriver 的更多使用详情可以去官方 [wiki](https://github.com/php-webdriver/php-webdriver/wiki) 或者去官方 [API](https://php-webdriver.github.io/php-webdriver/latest/) 文档上查阅。

最后用一个小例子结束这篇教程。







# END