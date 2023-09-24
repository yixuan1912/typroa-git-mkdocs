`nova-api` 服务:接受并响应最终用户的计算API调用, 它执行一些策略并启动大多数编排活动，例如运行实例。

`nova-compute` 服务: 通过守护程序API创建和终止虚拟机实例的辅助程序守护程序。

`nova-placement-api` 服务：跟踪每个提供商的库存和使用情况。

`nova-scheduler` 服务:  从队列中获取虚拟机实例请求，并确定它在哪台计算服务器主机上运行。

`nova-conductor` 模组:  `nova-compute`服务与数据库之间的中介, 处理两者的交互。

`nova-consoleauth` 守护程序：  为控制台代理提供的用户授权令牌。该服务必须正在运行，控制台代理才能起作用。

`nova-novncproxy` 守护程序：提供用于通过VNC连接访问正在运行的实例的代理。支持基于浏览器的`novnc`客户端。

`nova-spicehtml5proxy` 守护程序：提供用于通过SPICE连接访问正在运行的实例的代理。支持基于浏览器的HTML5客户端。

`nova-xvpvncproxy` 守护程序：提供用于通过VNC连接访问正在运行的实例的代理。支持特定于OpenStack的Java客户端。

`queue`: 在守护程序之间传递消息的中央中心。

SQL数据库:   存储云基础结构的大多数构建时和运行时状态，包括：

- 可用实例类型
- 使用中的实例
- 可用网络
- 专案

常见的数据库是用于测试和开发工作的SQLite3，MySQL，MariaDB和PostgreSQL。

