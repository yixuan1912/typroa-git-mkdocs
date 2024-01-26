



# ############# NAS 安装

https://www.bilibili.com/video/BV1Z14y1V7bW/?vd_source=528ef2cfdef1b47f8446a8da73e514c6

# ############# NAS 配置

## 首看
https://www.bilibili.com/video/BV1B3411s7Qi/?spm_id_from=333.788.recommend_more_video.0&vd_source=528ef2cfdef1b47f8446a8da73e514c6
https://www.bilibili.com/video/BV11Q4y1S7zr/?vd_source=528ef2cfdef1b47f8446a8da73e514c6

## ############# NAS 备份

https://www.bilibili.com/video/BV1jv411Y7fW/?spm_id_from=333.788&vd_source=528ef2cfdef1b47f8446a8da73e514c6
https://zhuanlan.zhihu.com/p/614820460
https://zhuanlan.zhihu.com/p/543970970


## 硬件
#### 主板  如果需要使用PCIE扩展槽的话，就不能买M.2槽位于主板底部的型号
七彩虹 CVN B660i GAMING  899 ：双网卡+wifi6 2个m2(一个背板)，四个sata3
技嘉 H270N 550：双网卡+wifi 1个m2，6个sata3

### cpu
i5 10400 780：2.9G、6c12t 核显65w
### 硬盘
西数 HC320 8T：1230
希捷   4T*2

2个16g内存

1个512的 .m2 做系统，一个 sata 做nas直通系统，四个sata做nas硬盘扩展 存储池

### 系统分配
pve
op 2g 2h
nas 6g 4h
centos(4青龙 fdd nonebot gocqhttp myweb fddweb) 12g 6h
精简版win11 6g 4h

三个日常容器
一个kedaya监控容器，存放所有自己的ck
一个所有ck的容器，存放自己的ck+万人骑ck，跑助力脚本，bbk，开卡等

![乔思伯N1配置](./img/%E4%B9%94%E6%80%9D%E4%BC%AFN1%E9%85%8D%E7%BD%AE.png)


magnet:?xt=urn:btih:d250c5c9ec4c44b8b54d28032d7d9fdf2b716c91&dn=%E7%BD%91%E7%BA%A2%E5%A5%B3%E7%A5%9E%E9%98%BF%E6%9C%B1%20%E8%A7%86%E5%9B%BE%E5%90%88%E9%9B%86&tr=udp%3A%2F%2Ftracker.openbittorrent.com%3A80&tr=udp%3A%2F%2Ftracker.opentrackr.org%3A1337%2Fannounce
magnet:?xt=urn:btih:00e8e8e2e8fd52a5bce09484f798f81405b535b7&dn=%E5%92%AC%E4%B8%80%E5%8F%A3%E5%85%94%E5%A8%98
magnet:?xt=urn:btih:14d4a12aacba7308ce23a72a900a424275a768db&dn=91-Pornhub--HornyCocoLove&tr=udp%3A%2F%2Ftracker.opentrackr.org%3A1337%2Fannounce&tr=udp%3A%2F%2Fopentracker.i2p.rocks%3A6969%2Fannounce&tr=https%3A%2F%2Fopentracker.i2p.rocks%3A443%2Fannounce&tr=udp%3A%2F%2Ftracker.openbittorrent.com%3A6969%2Fannounce&tr=http%3A%2F%2Ftracker.openbittorrent.com%3A80%2Fannounce&tr=udp%3A%2F%2F9.rarbg.com%3A2810%2Fannounce&tr=udp%3A%2F%2Fopen.demonii.com%3A1337%2Fannounce&tr=udp%3A%2F%2Fexodus.desync.com%3A6969%2Fannounce&tr=udp%3A%2F%2Fopen.stealth.si%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.torrent.eu.org%3A451%2Fannounce&tr=udp%3A%2F%2Ftracker.moeking.me%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker.bitsearch.to%3A1337%2Fannounce&tr=udp%3A%2F%2Ftracker1.bt.moack.co.kr%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.altrosky.nl%3A6969%2Fannounce&tr=udp%3A%2F%2Fmovies.zsw.ca%3A6969%2Fannounce&tr=udp%3A%2F%2Fexplodie.org%3A6969%2Fannounce&tr=https%3A%2F%2Ftr.burnabyhighstar.com%3A443%2Fannounce&tr=http%3A%2F%2Fopen.acgnxtracker.com%3A80%2Fannounce&tr=udp%3A%2F%2Fuploads.gamecoast.net%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker.theoks.net%3A6969%2Fannounce&tr=https%3A%2F%2Ftracker.bt4g.com%3A443%2Fannounce
magnet:?xt=urn:btih:078e8c2b7a944e51a6d15bf5590c2d4bc552aa0b&dn=%E8%A7%86%E9%A2%91-%E9%99%88%E6%B4%81%E8%8E%B9
magnet:?xt=urn:btih:8f27e6e47c8b521d5dd76ca77b252313bfc32d47&dn=%E5%9D%8F%E5%A7%90%E5%A7%90
magnet:?xt=urn:btih:905a2f57e6f681bda78467648638e03705f33b85&dn=11-20
magnet:?xt=urn:btih:ebf7cf44298b544d0f8622ca1aefdc98059e5cfd&dn=okirakuhuhu
magnet:?xt=urn:btih:b257c0fe5caf42537200b96e08618fdb808a88e1&dn=%5Bakibahonpo%5D%20NO%206357%20Miyuki%20%E3%81%BF%E3%82%86%E3%81%8D
magnet:?xt=urn:btih:00E8E8E2E8FD52A5BCE09484F798F81405B535B7
magnet:?xt=urn:btih:267f68aa087c495318f4020d02c7ac739931a054&dn=%E5%A4%A9%E6%B5%B4%20%E5%AE%8C%E6%95%B4%E7%89%88%E7%8F%8D%E8%97%8F%E9%AB%98%E6%B8%85%E7%89%88%E6%9C%AC%28%E6%9D%8E%E5%B0%8F%E7%92%90%29%28%E4%B8%AD%E6%96%87%E5%AD%97%E5%B9%95%29%281998%29.mkv&tr=udp%3A%2F%2Ftracker.publicbt.com%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.openbittorrent.com%3A80%2Fannounce

magnet:?xt=urn:btih:ca10fe19a44b6618beeb0c517668d88d0069c1bc&dn=[javdb.com]legsjapan-1109.torrent
magnet:?xt=urn:btih:1fe458e31e4527ff776d231f5278e05e4bf93478&dn=[javdb.com]shubao333@第一会所@LegsJapan - Yui Kasugano - Stockings Footjob
magnet:?xt=urn:btih:5c374b11cfb05792805159712b549388ce157083&dn=[javdb.com]081122_001-1pon-1080p
magnet:?xt=urn:btih:BA1FDA513C91D15911E56945799077D061819078