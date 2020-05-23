---
title: "AMD Renoir は HDMI 2.1 をサポート"
date: 2020-05-24T02:45:48+09:00
draft: false
tags: [ "Renoir", ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
---

足音をはっきりと立て、そろそろ姿を現しそうなデスクトップ向け[Renoir](/tags/renoir)だが、  
先日マザーボードベンダー各社から発表された AMD B550チップセット搭載製品の仕様から、HDMI 2.1 をサポートすることがわかった。  

## 8Kに対応するか、それとも未来のAPUに託すか

現行の AMD Navi1x世代GPU は、DisplayPort こそ DSC (Display Stream Compression) により 4K@240Hz、4K HDR@120Hz、8K HDR@60Hz に対応していたが、  
HDMI は ver2.0b に留まり、4K@60Hzまでとなっていた。  

> AMDの説明では，「HDMI 2.1は現状まだ不要である」というものだったが，Naviの設計段階で，HDMI 2.1対応のトランスミッタIPがなかった，というのが実情だろう。

> 引用元: <cite>[西川善司の3DGE：次世代GPU「Navi」の詳細とRadeon Softwareの新機能をレポート。操作遅延を低減する「Anti-Lag」は本当に効くのか？ - 4Gamer.net](https://www.4gamer.net/games/337/G033715/20190612140/)</cite>

Navi1x のディスプレイエンジン *DCN2.0* より進んだ、*Renoir* の *DCN2.1* なら DisplayPort、HDMI のどちらからも 8K出力が可能、と思いきや、  
B550チップセット搭載製品の仕様を見る限りでは、  
**GIGABYTE B550I AORUS PRO AX** [^1] はDisplayPort からは 5K@60Hz まで、HDMI からは 4K@60Hz、  
**ASRock B550 Phantom Gaming-ITX/ax** [^2] は DisplayPort 5K@120Hz、HDMI 4K@60Hz、  
**MSI MAG B550M MORTAR** [^3] は DisplayPort 4K@60Hz、HDMI 4K@24Hz となっており、  
対応解像度がバラバラではあるが、どれも 8Kには対応していない点では統一されている。(見落としていたら連絡下さると嬉しいです。)  
それと、恐らく MSI はまだデスクトップ向け *Renoir* の仕様を反映していないものと思われる。  

HDMI 2.1 が意味するのは、4K HDR と HDCP 2.3 への対応ということなのだろうか。HDMI 2.1 で追加された、可変リフレッシュレートの存在も考えられる。  
機能として *Renoir* の DisplayPort側は 8K解像度に対応しているはずだが、マザーボード側が対応していないとなると、  
AMD APU搭載機で 8K出力は次世代に託すこととなるかもしれない。  

[^1]: [B550I AORUS PRO AX (rev. 1.0) | Motherboard - GIGABYTE Global](https://www.gigabyte.com/Motherboard/B550I-AORUS-PRO-AX-rev-10/sp#sp)
[^2]: [ASRock > B550 Phantom Gaming-ITX/ax](https://www.asrock.com/mb/AMD/B550%20Phantom%20Gaming-ITXax/index.asp#Specification)
[^3]: [Specification for MAG B550M MORTAR | Motherboard - The world leader in motherboard design | MSI Global](https://www.msi.com/Motherboard/MAG-B550M-MORTAR/Specification)

余談だが、Intel Z490チップセット搭載製品の仕様から、[Rocket Lake](/tags/rocket_lake)で DisplayPort 1.4 に対応すると見られるが、HDMI は ver1.4のままとなっている。  
{{< link >}}[Rocket Lakeの噂、そこからの推測 | Coelacanth's Dream](/posts/2020/05/04/rocketlake-rumor-guess/#rkl-gpu){{< /link >}}
このあたり、間接的に統合GPUの仕様で牽制しているようにも見えて面白い。  

また、HDMI 2.1 はまだ普及しているとは到底言えず、対応しているのは高級テレビばかりだが、  
次世代コンソール機、PlayStation5 と Xbox Series X の両方が HDMI 2.1 に対応しているため、それにより普及の加速とゲーミングモニターへの搭載が期待できる。  

{{< ref >}}

 * [Complete List of Televisions with HDMI 2.1 in 2020 | PremiumBuilds](https://premiumbuilds.com/guides/hdmi-2-1-tv-list/)

{{< /ref >}}
