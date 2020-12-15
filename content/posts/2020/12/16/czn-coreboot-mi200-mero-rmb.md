---
title: "Cezanne の Coreboot 対応が進行中　―― MI200, Mero, Rembrandt のネームプレート"
date: 2020-12-16T05:08:29+09:00
draft: false
tags: [ "Cezanne", "VanGogh", "Mero", "Rembrandt" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU", "GPU" ]
noindex: false
# summary: ""
toc: false
---

*Zen 3 + GFX9/Vega* の構成を取る AMD 次世代 APU *Cezanne* だが、現在 Coreboot の対応、それに合わせて Google Chromebookボードの開発も進められている。  
{{< link >}} [topic:"cezanne" (status:open OR status:merged) · Gerrit Code Review](https://review.coreboot.org/q/topic:%22cezanne%22+(status:open%20OR%20status:merged)) {{< /link >}}
そして、Coreboot のサポートにおいてオープンではない部分、PSP (Platform Security Processor) や VBIOS のファームウェアを提供する [amd/firmware_binaries](https://github.com/amd/firmware_binaries) にも Cezanne 関連のファイルがアップロードされた。  

*Cezanne* の VBIOSリリースノートには DeviceID `0x1638` が記述されており、以前 Coreboot に追加された `Family: 19h, Model: 51h` の DeviceID と一致する。  
{{< link >}} [Coreboot に Renoir と Lucienne/Cezanne らしき DeviceID が追加される | Coelacanth's Dream](/posts/2020/11/19/rn-lcn-czn-coreboot-did/) {{< /link >}}

 >     Cezanne A0 0x1638 105_D27000_00A  CZNGenericVbios.009  07/01/20,02:03:02 2139194@17.10.0.21  ATOMBuild#520539

Geekbench の実行結果で *Cezanne* は `Family: 19h(25), Model: 50h(80)` となっていたが、`Stepping 0` であることも合わせ、やはりそれは仮のものだったのかもしれない。  
{{< link >}} [Cezanne APU、Ryzen 7 5800U が Geekbench に現る | Coelacanth's Dream](/posts/2020/11/26/r7-5800u-czn/) {{< /link >}}

## 拾ったネームプレート

今回公開された、PSP や VBIOS といったクローズドなバイナリから読み取れることは少ない。リバースエンジニアリング、逆アセンブル等もライセンスで禁止されている。  
読み取れるのは精々ファイル名から、*Cezanne* は *Renoir* 同様 DDR4/LPDDR4x をサポートし、ディスプレイエンジンに DCN2.1 が搭載されているということくらいだ。  
ただ {{< comple >}} 呆れに近いものを覚えるが {{< /comple >}} クローズドなバイナリに混じって更新履歴を記したテキストファイルも一緒に公開されていた。  
[MI100/Arcturus](/tags/arcturus) の後継であろう **MI200** や、*Mero (MR)* 、*Rembrandt (RMB)* といった名前がある。他には *Badami* だとか  
*Mero* は *VanGogh* と一緒に名前が出ていることが多く、[Dali](/tags/dali) と [Pollock](/tags/pollock) のような関係にあるのかもしれない。  
**MI200** は SMU のバージョンが、これまたひょっこり現れた [Yellow Carp](/tags/yellow_carp) APU と同じ SMU v13 となるような記述があった。  

ただ自分が拾ったのは簡素なネームプレートのようなもので、ドッグタグ程その名前の持ち主の詳細が記載されている訳ではない。それらの魅力はもっと先の所にあると知っている。見え始めるのはオープンソースドライバー、Linux Kenrel や Mesa3D に堂々とパッチが投稿された時だ。  

