## overview

1. openwrt-deploy

2. plugins

    - openclash
    - adguardhome(todo)
    - frp(todo)

3. proxy

4. 疑难杂症

## 1. openwrt-deploy


部署指导

- https://sspai.com/post/68511
- https://tanweime.com/2023/05/03/%E6%A0%91%E8%8E%93%E6%B4%BE3b%E6%90%AD%E5%BB%BAopenwrt%E7%A7%91%E5%AD%A6%E4%B8%8A%E7%BD%91/
- https://zhuanlan.zhihu.com/p/112484256
- https://zhuanlan.zhihu.com/p/451788328

    里面一些踩坑经验还是挺有用的。根据指导，选择了魔改OpenWrt系统：ImmortalWrt，[下载链接](https://downloads.immortalwrt.org/)

- firmware: https://github.com/openwrt/openwrt/tree/master/target/linux/ar71xx/image


## 2. plugins

插件安装

### 2.1 OpenClash

- [OpenClash官方Wiki](https://github.com/vernesong/OpenClash/wiki/%E5%B8%B8%E8%A7%84%E8%AE%BE%E7%BD%AE)
- https://www.p-chao.com/2024-02-09/%E8%BD%AF%E8%B7%AF%E7%94%B1%EF%BC%88%E4%B8%83%EF%BC%89openclash%E5%AE%89%E8%A3%85%E5%92%8C%E9%85%8D%E7%BD%AE/
- https://adao.me/ruanjian/216.html

### 2.2 AdGuardHome(todo)

- https://www.cnblogs.com/osnosn/p/17046580.html
- https://mxcheats.com/87.html
- https://post.smzdm.com/p/ar670qog/

### 2.3 Frp(todo)

- https://segmentfault.com/a/1190000044576121?utm_source=sf-similar-article

## 3. proxy

- [浅谈在代理环境中的 DNS 解析行为](https://blog.skk.moe/post/what-happend-to-dns-in-proxy/)
- [旁路由原理与配置](https://easonyang.com/posts/transparent-proxy-in-router-gateway/)
- [clash fake ip 模式是否可以解决 dns 污染？ - v2ex.com](https://www.v2ex.com/t/838687)

## 4.疑难杂症

记录OpenWrt使用过程中遇到的问题

1. 无法访问Azure远程服务器（未解决）

背景：Ubuntu机器连接TPLink路由器（主路由），网关和DNS配置OpenWrt路由器。终端（Ubuntu）无法访问Azure远程服务器。