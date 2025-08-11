title: 这一点也不好玩——关于我土法酿制了一个OJ这回事
categories: 
tags: []
date: 2022-04-27 11:06:00
---
折腾。

<!--more-->

# 0.Beginning

luxu：最近信息课去听课了一下，感觉用 OJ 教学的效果不错？我们那个[原有的OJ](https://oj.doubeecat.xyz/)能不能用上呢？

我：可以吧你试试

（一节课试验失败后）

我：![image-20220427184223091](https://s2.loli.net/2022/04/27/2x5rKFoRPBshv9a.png)

# 1.折腾的开始

当时面对的需求：

- 批量添加用户
- 作业，训练计划

但是这些 syzoj 都没有！

有同学可能会问了：你不能自己写一个吗？

我觉得可行，但是当初偷懒用 docker 搭建了 syzoj，所以寄。

然后想起来，有人推荐过 [Hydro](https://hydro.js.org/) ，看了下功能感觉很符合需求，而且安装还快！所以决定使用 Hydro.

发现安装很简单！只要输入一行安装脚本即可！

 ![image-20220427184914631](https://s2.loli.net/2022/04/27/DE5S7dCfYmb8rXT.png)

直到我发现，正常人搭服务器都会用 Ubuntu 的 LTS 版，只有我在用发行版。

所以折腾开始了。

# 2.MongoDB

非常开心输入了安装脚本，但是发现没法运行。

![image-20220427185122463](https://s2.loli.net/2022/04/27/Fq7TJNPcWC8huGm.png)

安装脚本会按照版本号自动匹配，但是镜像里的支持版本都是 LTS，所以脚本寄。

于是开始了手动安装的过程，第一步是安装 MongoDB。

年轻的 Doubeecat 涉世未深，盲目相信了 `apt-get` 是一条正道，于是他 `apt-get install mongodb`

好了！装上了！

```bash
> mongo
> db.version()
3.6.2
```

我日。

我日。

我日。

好，那就换版本吧，`uninstall` ，然后找官方镜像，然后安装，很好，很正常，一点问题都没有……

个屁啊。

```bash
> mongod
> Failed to start up WiredTiger under any compatibility version. This may be due to an unsupported upgrade or downgrade.
> Stopping now...
```

我日，这是啥jb问题。

上网查，无果。定位问题在 `WiredTiger` ，但是没有思路，群友建议。

![image-20220427185750790](https://s2.loli.net/2022/04/27/Ct1I98NrX4zjdlh.png)

困难的。

第二天早上醒了，起床之后想到是不是之前 `apt-get` 安装的时候没有完全卸载，然后手动清了 `/data` 目录，我日，还真可以。

![image-20220427185903210](https://s2.loli.net/2022/04/27/McTykZAEWSInsH6.png)

很激动，虽然代价是我舍弃了周日早上的睡眠。

# 3.折腾

很开心，于是尝试 debug 模式启动 web 端

![image-20220427190011394](https://s2.loli.net/2022/04/27/FhkOR6ugVmWjITb.png)

好，发现 minio 没接上，小问题！

![image-20220427190036092](https://s2.loli.net/2022/04/27/5qZJ6AuNtTapiGO.png)

好，发现配置文件写错了，小问题！

![image-20220427190108182](https://s2.loli.net/2022/04/27/mBEKhMSDbjdx2i6.png)

好，发现还是没写对，小问题！

![image-20220427190155307](https://s2.loli.net/2022/04/27/yvB4FeV1Em76T39.png)

DB 没通，大问题。

冷静了下，发现忘记改配置文件了，小问题！

![image-20220427190221467](https://s2.loli.net/2022/04/27/uwdC6Wo38yA4g9b.png)

**厦大附中 OnlineJudge 上线啦！**

# 4.运维

段子一则：

> Doubeecat 醒来，感觉今天又是美好的一天呢！
>
> 点开 QQ。
>
> luxu：cat，OJ怎么又挂了！
>
> 感觉今天又是灾难的一天呢！

![image-20220427190429093](https://s2.loli.net/2022/04/27/UuqoW49pvRmzQ3N.png)

接下来几天就是，发现bug-> 内网穿透挂了 or db挂了 -> debug.... -> 等待下一个bug的到来。

总之就是折腾，希望我的这篇碎碎念能带来一些教训。