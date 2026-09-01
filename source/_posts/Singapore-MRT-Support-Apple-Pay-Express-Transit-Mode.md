---
title: 新加坡 MRT 支持 Apple Pay 快捷公交卡功能了
date: 2026-09-01 22:08:25
categories:
- Information
tags:
- Information
- Singapore
- MRT
- Apple Pay
---

昨天刷到 Reddit 上讨论新加坡 MRT 部分站点支持 Apple Pay 快捷公交卡功能了，具体可参考[这个](https://www.reddit.com/r/singapore/comments/1w2favz/apple_pay_express_transit_mode_works_at_some_mrt/)帖子。于是下班时试了下，真的可以了。

根据 Reddit 的讨论，目前这个功能应该还在早期测试，不是所有 MRT/LRT 站点都支持，巴士目前还不支持。但根据交通部长的说法，年底前应该会全部支持。

<!-- more -->

正好借此机会，简单聊一下 Apple Pay 这个[快捷公交卡](https://support.apple.com/en-us/105123)功能。

快捷公交卡功能可以在系统设置的 `Wallet & Apple Pay` - `Express Transit Card` 里开启，目前支持交通卡和支付卡两种。

前者主要是 Apple Pay 里支持添加的公交卡，包括中国大陆的交通联合卡、香港的 Octopus、日本的 Suica/PASMO/ICOCA/TOICA、韩国的 Tmoney、法国的 Navigo、加拿大的 PRESTO 和美国的若干公交卡。

后者则是受支持的 Visa/Mastercard 银行卡，像新加坡这种没有公交卡支持的，可以直接刷 Visa/Mastercard。

![](/images/Singapore-MRT-Support-Apple-Pay-Express-Transit-Mode/1.jpg)

早期的快捷公交卡功能应该是只支持公交卡的，并且可以开启多张，但部分卡会互斥（比如中国大陆不同地区的交通联合卡只能开启一张）。开启后，在刷卡时不用双击电源键进入 Apple Pay 选卡，直接把手机放到读卡器上即可自动读取信息。之前我在国内的时候，就是把上海公交卡和日本 Suica 都开启，这样国内和日本都可以锁屏直接进出站了。

然而，Apple Pay 不支持新加坡这边的公交卡，只能通过双击电源键进入 Apple Pay 选择 Visa/Mastercard 卡，这一步需要 Face ID 验证。几年前，快捷公交卡支持了支付卡，对于支持的地区（我也不知道有哪些地方）也可以在锁屏状态下直接刷卡进出站，会自动读取选中的 Visa/Mastercard 卡。但是新加坡当时并不支持，我之前试过同时开启上海公交卡、日本 Suica 和 Visa/Mastercard 支付卡时，锁屏放到读卡器上会弹出上海公交卡，并读取失败。

昨天开始后，我还是像上面说的同时开启上海公交卡、日本 Suica 和 Visa/Mastercard 支付卡，这次锁屏放到读卡器后，马上就提示 Visa 读卡成功，读卡器上显示 BANK CARD。这也说明当读卡器支持快捷模式时，如果当地没有支持的 Apple Pay 公交卡，会优先读取设置的 Visa/Mastercard 支付卡，而不会触发其他公交卡的读取。

总之，希望这个功能赶快全量吧，特别是巴士早日支持。
