---
title: "ブロックチェーン向けの Navi10 SKU とそこから読める文脈"
date: 2020-10-21T17:39:22+09:00
draft: false
tags: [ "Navi10", ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
# summary: ""
toc: false
---

Linux Kenrel (amd-gfx) にブロックチェーン向けの *Navi10* SKU をサポートするパッチが投稿された。  
{{< link >}} [[PATCH 1/3] drm/amdgpu: add DID for navi10 blockchain SKU](https://lists.freedesktop.org/archives/amd-gfx/2020-October/055016.html) {{< /link >}}
DID (DeviceID) はこれまでに使用されていなかった `0x731E` が使われ、RID (RevisionID) には `0xC6` と `0xC7` の 2つが用意されており、ブロックチェーン向けの SKU もまた 2種類存在すると思われる。  
ディスプレイコントローラー (DCN2.0) とマルチメディアエンジン (VCN2.0) はサポートされず、ドライバーからは無効化される。  
ブロックチェーン/マイニング向けの AMD GPU と言うと、いつかに話題になった映像出力が実装されていない中古の格安 **RX 470** が日本に流れてきたのを思い出す。[^mining-rx470]  
その時は自力で映像出力を実装する猛者が現れたりしたが、今回のは残念ながらドライバーから無効化されている。それ用の DID/RID が用意されているあたり、AIB、カードベンダーが独自に既存のカードからただ映像出力を省いた訳ではないように思える。  

[^mining-rx470]: [在庫は大量、マイニング向けの格安Radeon RX 570/470が一部ショップに再入荷 （取材中に見つけた○○なもの） - AKIBA PC Hotline!](https://akiba-pc.watch.impress.co.jp/docs/wakiba/find/1165171.html)

さて、同じく *Navi10* をベースとするコンシューマ向けの **RX 5700/XT** には今月 (2020/10) の初めに近く生産終了 (End Of Life) し、AIB仕様もそれに沿うという話が出てきた。[^rx-5700-eol]  
その後、話が更新され、**RX 5700 XT** は 2020Q1 まで製造されるが、対し **RX 5700** は実際に流通が減少しており、特別な注文には対応するとのことだった。  

[^rx-5700-eol]: [AMD Radeon RX 5700 series reach end of life - VideoCardz.com](https://videocardz.com/newz/amd-radeon-rx-5700-series-reach-end-of-life) <br> [Les cartes graphiques AMD RADEON RX 5700 Custom sont passées en fin de vie - Cartes graphiques](https://www.cowcotland.com/news/73563/les-cartes-graphiques-amd-radeon-rx-5700-custom-sont-passees-en-fin-de-vie.html)

そして今月の中頃には、AIBパートナーである XFX が 2020/09 中、AMD GPUチップをコンシューマ向け製品に使わず、ブロックチェーン/マイング向け製品にほとんど使ったことを中国のニュースサイト [MyDrivers](https://www.mydrivers.com/) が報じている。[^xfx-mining-card]  
**RX 580, RX 590, RX 5600 XT, RX 5700 XT** 等の製品はその影響で在庫切れになったとのこと。  

[^xfx-mining-card]: [AMD显卡骚操作：不零售 直接卖给挖矿的！-AMD,显卡,挖矿,矿主,讯景,XFX ——快科技(驱动之家旗下媒体)--科技改变未来](https://news.mydrivers.com/1/718/718219.htm) <br> [XFX China allegedly sells 'almost all' available Radeon RX stock to mining farms - VideoCardz.com](https://videocardz.com/newz/xfx-china-allegedly-sells-almost-all-available-radeon-rx-stock-to-mining-farms)

今回 Linux Kernel (amd-gfx) へのパッチ投稿で明かされたブロックチェーン向け *Navi10* SKU はこの話の 1つの裏付けになるのではないかと思う。  
さらに考えれば、{{< comple >}} AMD と AIBパートナーどちらが先に要望したのかはさすがに分からないが  {{< /comple >}} 再度仮想通貨の値が高まり、ブロックチェーン/マイニング向けに GPU の需要もまた高くなると見込み、軌道をわずかに変更させ、GPUチップをブロックチェーン/マイニング向け製品に使用、結果コンシューマ向けの **RX 5700** の流通が減少、生産終了となった、という文脈が読めてくる。  

AMD は *RDNA 2* GPU を 2020/10/28 に発表する。  
前世代 GPUチップの投入先を切り替えるにはちょうど良いタイミングだったのかもしれない。それにより、新製品となる *RDNA 2* GPU が {{< comple >}} かつての **Radeon Vega56/64 のように** {{< /comple>}} マイニング需要に巻き込まれることを緩和できるだろう。  
ただそうなると、*RDNA 2* GPU がどれだけ確保されているかが、市場においてより重要となりそうだが果たして。  
