---
title: "Navy Flounder/Navi22 のダイ観察"
date: 2021-12-02T02:50:24+09:00
draft: false
tags: [ "DieShot", "Navy_Flounder", "RDNA_2" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
---

[Fritzchens Fritz](https://www.flickr.com/photos/130561288@N04/) 氏により、*RDNA 2 アーキテクチャ* を採用する [Navy Flounder/Navi22](/tags/navy_flounder) のダイショットが公開された。  
Intel は最近 CPU のダイショットを積極的に公開しているようには感じるが、研究発表の場以外では大体の場合において、内部構造を模した CG か一部を置き換えた、あるいはぼかしたものがダイショットの代わりに使われることが多い。そのため、スキルある人が得たデータを広くに公開してくれることはは貴重である。  

 * [AMD@7nm@RDNA_2nd_gen@Navi22@Radeon_RX_6700_XT@215-0932396@… | Flickr](https://www.flickr.com/photos/130561288@N04/51703830446/)

氏の測定によれば、*Navy Flounder/Navi22* のダイサイズは 334.980mm2、そう変わらないが **RX 6700 XT** 発表時に AMD が公表したダイサイズは 336mm2 だった。[^die-size]  

[^die-size]: [西川善司の3DGE：Radeon RX 6700 XTの性能は，「Infinity Cache」が鍵を握る](https://www.4gamer.net/games/461/G046171/20210316071/)

## Infinity Cache/MALL/L3キャッシュ {#mall}

RDNA 2 dGPU から導入された、マーケティング的には *Infinity Cache* 、オープンソース・ドライバー中では *MALL (Memory Access Last Level)* 、キャッシュ階層で言えば L3キャッシュに当たるそれは、*Navy Flounder/Navi22* には 96MiB 搭載されている。  
ダイショットにおいて L3キャッシュブロックはかなり目立ち、上下部に敷き詰めるようにして配置されている。  

*Navy Flounder/Navi22* のメモリインターフェイスは GDDR6 192-bit (16-bit x12ch)、メモリチャネルあたりの L3キャッシュサイズは 8MiB となるが、ダイショットでは 16ブロック確認でき、ブロックあたり 4MiB の実装となっている。  

*Navy Flounder/Navi22* より小規模な構成を採る RDNA 2 dGPU、[Dimgrey Cavefish/Navi23](/tags/dimgrey_cavefish)、[Beige Goby/Navi24](/tags/beige_goby) は、メモリチャネルあたりの L3キャッシュが 4MiB となっており、ブロックあたりのサイズと一致する。  
それらにおいては、おそらくブロック (4MiB) の設計は基本そのままに、メモリチャネルあたり 1ブロックとなるような実装になっているのではないかと思われる。  
{{< link >}} [Dimgrey Cavefish/Navi23 の Infinity Cache はメモリチャネルあたり 4MiB | Coelacanth's Dream](/posts/2021/03/04/dimgrey_cavefish-4mb-mall-per-ch/) {{< /link >}}
{{< link >}} [新たな RDNA 2 GPU、「Beige Goby」 をサポートするパッチが投稿される | Coelacanth's Dream](/posts/2021/05/13/amd-beige_goby/#cache) {{< /link >}}

また GPU L3キャッシュは、[VanGogh](/tags/vangogh)、[Yellow Carp/Rembrandt](/tags/yellow_carp) といった *RDNA 2 アーキテクチャ* を採用する APU では搭載されていないことが明らかにされている。  
{{< link >}} [AMD GPU のキャッシュ構成情報　―― Dimgrey Cavefish / Aldebaran / VanGogh | Coelacanth's Dream](/posts/2021/03/30/amdgpu_cache_info/#vgh) {{< /link >}}

| RDNA 2 | Sienna Cichlid/Navi21 | Navy Flounder/Navi22 | Dimgrey Cavefish/Navi23 |
| :-- | :--: | :--: | :--: |
| WGP (CU) | 40 (80) | 20 (40) | 16 (32) |
| Memory bus width | 256-bit | 192-bit | 128-bit |
| L3 cache | 128MiB | 96MiB | 32MiB |
| Transistor count | 26.8M | 17.2B | 11.1B |
| Die size | 519mm2 | 336mm2 | 223mm2[^w6600-diesize] |

[^w6600-diesize]: [AMD、Radeon Pro 6000Wシリーズを発表 - RDNA 2世代の「W6600」と「W6800」 | マイナビニュース](https://news.mynavi.jp/article/20210608-1901242/)

WGP 20基 (CU 40基) というくくりでは、*Navi10* と同じ。  
*Navy Flounder/Navi22* は *Navi10* から、RB+ の有効化を前提としたことにより RB+ の実装面積を削減 (16基 -\> 8基)、メモリバス幅の削減 (256-bit -\> 128-bit) といったダイサイズを節約できる要素があるが、  
{{<  link >}} [一部の AMD GPU で実装、有効化されている RB+ とは何か | Coelacanth's Dream](/posts/2020/11/10/what-is-rbplus/) {{< /link >}}
L3キャッシュ追加の影響が大きく、結果としてダイサイズは約 1.34倍となっている。(Navi10: 251mm2)  

### L3キャッシュ 比較 {#compare-l3c}

RDNA 2 L3キャッシュは *Zen アーキテクチャ* のデザインをベースにしていることが AMD より語られており、それによって高密度、高クロック動作を可能にしているとする。  

以下は *Navy Flounder/Navi22* の L3キャッシュブロック (4MiB) と、*Zen 3 アーキテクチャ* の L3キャッシュスライス (4MiB) を並べた画像。  
*Zen 3* のダイショットも [Fritzchens Fritz](https://www.flickr.com/photos/130561288@N04/) 氏が撮影したものを用いている。  

 * [AMD@7nm(12nmIOD)@Zen3@Vermeer@Ryzen_5_5600X@100-000000064_… | Flickr](https://www.flickr.com/photos/130561288@N04/50579552573/)

{{< figure src="../navy_flounder-zen_3-4mb.webp" title="4MB Cache: Navy Flounder/Navi22・Zen 3" caption="画像出典: <br> [AMD@7nm@RDNA_2nd_gen@Navi22@Radeon_RX_6700_XT@215-0932396@… | Flickr](https://www.flickr.com/photos/130561288@N04/51704509579/), <br> [AMD@7nm(12nmIOD)@Zen3@Vermeer@Ryzen_5_5600X@100-000000064_… | Flickr](https://www.flickr.com/photos/130561288@N04/50579552573/)" >}}

*Zen 3* L3キャッシュには LDO (Low-Dropout regulator) と V-Cache のための TSV部が含まれている。  
そうした点もあるが、*Navy Flounder/Navi22* ではタグが *Zen 3* よりも少ない構成となっている。タグは *Navy Flounder/Navi22* では下部、*Zen 3* L3キャッシュでは中央左右に配置されている。  

RDNA系アーキテクチャではキャッシュラインサイズが 128Byte であること、RDNA L3キャッシュは各メモリチャネルに対応したメモリサイドキャッシュだと考えられ、キャッシュコヒーレンシを維持する機構を必要としないことがより高密度な実装に繋がっているのではないかと思われる。  

| L3 4MiB | *RDNA 2* | *Zen 3* |
| :-- | :--: | :--: |
| Area | ~2.474mm2 | ~4.242mm2 |

## WGP 比較 {#compare-wgp}

{{< figure src="../navy_flounder-navi14-ps5-2wgp.webp" title="2 WGP: Navy Flounder/Navi22 ・ Navi14 ・ PS5" caption="画像出典: <br> [AMD@7nm@RDNA_2nd_gen@Navi22@Radeon_RX_6700_XT@215-0932396@… | Flickr](https://www.flickr.com/photos/130561288@N04/51703830446/), <br>[AMD@7nm@RDNA_1th_gen@Navi14@Radeon_RX_5500_XT@215-0932396@… | Flickr](https://www.flickr.com/photos/130561288@N04/49437016132/), <br> [AMD@7nm@Zen2_RDNA_APU@Oberon@PlayStation5@100-000000189_9J… | Flickr](https://www.flickr.com/photos/130561288@N04/50951750013/)" >}}

向かい合うように配置された WGP 2基を、*Navy Flounder/Navi22・Navi14・PS5* から切り出し、並べたのが上の画像。  
サイズは実サイズに合わせたが、大体であり、あまり自信はない。  

*Navi14・PS5・Xbox Series X* で比較したとき、*PS5・XSX* は同様の WGP レイアウトであることが見て取れたが、今回はそれぞれで部分的に異なっている。  
{{< link >}} [PS5 のダイ観察 | Coelacanth's Dream](/posts/2021/02/15/ps5-dieshot/) {{< /link >}}
また *Navy Flounder/Navi22* と *Navi14* の比較において、WGP のエリアサイズはそれほど変わっていないように思う。  
WGPレベルでの *RDNA* から *RDNA 2* の変更点には、レイトレーシング対応と、SIMDユニットあたりの Waveバッファを 20エントリから 16エントリに減らしたことが挙げられる。(ドット積命令に関しては *Navi14* との比較では変わらない)  
逆に、製造プロセスが変わらないからこそ WGPレベルでは大きな変更を取り入れなかったのかもしれない。  

また、実サイズに即している自信がない理由でもあるのだが、*PS5* の WGP は他 2つと比べて明らかに小さい。  
WGP 2基を並べた中心に位置する、TMU (Texture Mapping Unit) + RA (Ray Accelerator) 4基のレイアウトは、*PS5・XSX* では L字ブロックを組み合わせたような配置になっている。  
それ以外の部分、SIMD32ユニット 4基や LDS などのメモリもロジック部が小さくなっているように見える。  

| WGP 2-units | compare with Navi14 |
| :-- | :--: |
| *Navy Flounder* | x1.02 |
| *Navi14* | x1.00 |
| *PS5* | x0.92 |

