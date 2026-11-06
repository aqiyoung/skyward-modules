# skyward-modules

给 [Skyward](https://github.com/aqiyoung/Skyward) 用的规则模块。Surge 的 `.sgmodule` 格式，同一个文件也能直接给 Shadowrocket、Loon、Stash 用。

模块里只有域名规则，没有 URL 重写、没有脚本、不碰 MITM，所以在 mihomo 内核上可以完整生效——导入后不会有一堆"已跳过"。

## 模块

| 模块 | 内容 | 导入链接 |
|---|---|---|
| 去广告 | 广告联盟、统计与归因、崩溃上报，约 120 条根域 | [raw](https://raw.githubusercontent.com/aqiyoung/skyward-modules/main/modules/ads.sgmodule) · [jsDelivr](https://cdn.jsdelivr.net/gh/aqiyoung/skyward-modules@main/modules/ads.sgmodule) |

国内直连 `raw.githubusercontent.com` 不稳时用 jsDelivr 那条，内容一样。

## 导入

Skyward：**进阶配置 → 模块 → 右上角加号 → 从链接导入**，粘贴上面的链接。

导入后模块出现在列表里，带一个总开关，可以拖动排序。点进去能看到它包含的规则、声明的参数，以及被跳过的部分——这个模块的"已跳过"应该是空的。

规则会插在订阅自带规则之前，所以它能压住订阅里的分流；不想要时关掉开关即可，不需要删订阅。

链接导入的模块可以长按选 **更新**，会回到原地址重新下载。

## 覆盖范围

- **广告联盟**：穿山甲、广点通/优量汇、百度联盟、阿里妈妈/TanX、快手联盟、小米/OPPO/vivo/三星的广告 SDK，以及多盟、品友、有米、万普、点乐这些国内平台
- **自建广告系统**：微博的 `uve.weibo.com`（开屏预加载与素材）、`adstrategy.biz.weibo.com`（投放策略）、新浪 SAX 广告交易域
- **海外广告网络**：AppLovin、Unity Ads、ironSource、Vungle、Chartboost、AdColony、InMobi、Amazon Ads、DoubleClick、AdMob 相关域、Criteo、Taboola、Outbrain 等
- **统计与归因**：友盟、TalkingData、神策、GrowingIO、诸葛、易观、热云、CNZZ、51.LA、百度统计、腾讯 MTA/灯塔、AdMaster、秒针、Google Analytics、Firebase Analytics、Adjust、AppsFlyer、Branch、Kochava

全部是 `DOMAIN-SUFFIX` 匹配的根域，子域自动覆盖。

## 开屏广告为什么有时拦不住

这是域名拦截的固有边界，不是模块漏了域名：

1. **素材下发了才拦**——广告位由主 API（如 `api.weibo.cn`）随内容一起下发，封掉素材域只是让图片加载失败，界面上可能变成白屏或灰块，仍然是那三秒
2. **本地有缓存**——开屏素材会被 App 缓存，规则生效前已下载的那一条照样播完
3. **不经过第三方**——大厂自己的广告系统跑在自家域下，通用联盟列表一个都不沾，只能逐个点名

所以微博这类，加规则后要**在系统设置里清掉微博的缓存**（或者卸载重装），否则新规则对已经缓存的素材无效。

判断到底有没有生效，看 Skyward 的连接日志：过滤 `weibo`，如果 `uve.weibo.com` 下的请求显示 `REJECT`，说明规则在起作用，剩下的就是客户端自己把空白页播完。

## 刻意没做的

- **推送**：个推、极光、友盟推送这些 SDK 也上报数据，但封了会收不到消息，不算在内
- **登录与支付**：风控、设备指纹类域名（数美、同盾等）一旦误封会导致登录失败或支付中断，一个都不碰
- **URL 重写、脚本、MITM**：mihomo 不做 HTTPS 解密，也没有 JavaScript 运行时，写进来只会让模块多出一堆无法生效的段落

## 可能的误伤

- `e.qq.com` 是广点通的点击跳转域，个别腾讯系页面的分享落地页也会经过它
- `app-measurement.com` 是 Firebase Analytics，少数 App 拿它做远程配置，封了可能影响 A/B 实验或开关下发
- `tui.weibo.com` 和 `alitui.weibo.com` 是微博信息流的推广位，封掉后信息流里带"推广"标记的微博也会一起消失
- 厂商广告域（`ads.oppomobile.com`、`adv.sec.miui.com` 等）关闭后，系统自带应用的个性化推荐会失效，功能本身不受影响

遇到哪个 App 不正常，把对应的一行删掉再更新模块即可。

## 自己改

Fork 之后改 `modules/ads.sgmodule`，格式就是 Surge 模块：

```
#!name=名字
#!desc=说明
#!author=作者
#!version=版本
#!homepage=主页

[Rule]
DOMAIN-SUFFIX,example.com,REJECT
```

`#` 开头是注释，空行随意。规则目标写 `REJECT` 或 `REJECT-DROP`；写 `PROXY` 这类模块自己的关键字，Skyward 会要求你在详情页指定一个实际策略。

## 许可

MIT
