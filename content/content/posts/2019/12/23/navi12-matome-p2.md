---
title: "Navi12 Matome Part2"
date: 2019-12-23T21:51:54+09:00
draft: false
tags: [ "Radeon", "RDNA", "Navi12", "GFX10", "gfx1011" ]
keywords: [ "Radeon", "Navi12" ]
categories: [ "Hardware", "AMD", "GPU" ]
---

そろそろ出てきてよさそうなNavi12だが、ここに来て情報が錯綜してきた。  
Navi21が近いと囁かれていることからキャンセルされるんじゃないかとか、RX 5600シリーズに使われるのがNavi12じゃないかとか。  
他にも矛盾する噂が存在してたりと、噂が飛び交うのが常である半導体業界だがNavi12に限っては特にそうだ。  

そして次に聞こえてくるのはハテナマークだけが浮かぶ与太話か、矛盾そのままを抱えたカオスか、悲観的な妄想。  
自分がそれを発信しない人間だとは言い切れないが、そうでないようには努めたい。  
そのため、Linux KernelやMesa3Dといったオープンソースからだけの情報を載せていく。  

<code>過去まとめ: [Navi12 Matome](/posts/2019/11/05/navi12-matome/)  </code>

### Navi12のGameClockは1100MHz?
Linux Kernelへのパッチから。  
[[2/2] drm/amdgpu/smu: add peak profile support for navi12 - Patchwork AMD X.Org drivers](https://patchwork.freedesktop.org/patch/346277/)  

あくまでボード側のVBIOSにPPTableのプロファイルが無かった場合に読み込むものだが、  
これまでのNavi10/14の例で言えば、1100MHzがNavi12のGameClockとなる。  

#### ここから推測

が、疑問点が2つあり、1つはGameClockが低すぎるというもの。  
Navi1x系列では最も低く、Navi14 XLM（RX 5300M）よりも低い。  
もう１つは

 > NAVI12_UMD_PSTATE_PEAK_GFXCLK     (1100)

とだけで、XTXやXLといったSKUを示すような記述がないことだ。  
Navi12のSKUが1種類だけ（= リビジョンが1つだけ）ならば成り立つが、やはり不自然、と言うかイレギュラーだと思わされる。  

半導体には歩留まりがあり、ダイのスペック通りのものだけが出る訳がないため、１種類のダイで1種類のSKUなんてのは通常滅多にやらない。やるとしてもほとんどはゲーム機のためであり、そこでは冗長部分を加えたり仕様クロックを低めにすることで多くのダイが基準をクリアできるようにする。  

だがNavi12関連のコードがLinux Kernel、Mesa3Dに追加されていることから、ゲーム機なんてことはなくコンシューマー市場に出てくると見ている。  
VF(Virtual Function)用のDeviceIDもあることから、サーバー向けGPUとしても出てくるはずだ。  

そのため1100MHzというのは仮のクロックな可能性もある。  

***もしも、*** GameClock:1100MHzが正しいとすると、あり得るのはそれだけクロックを下げないといけないくらいWGPが多い（= ダイサイズが大きい）か、MCM（Multi-Chip Module）構成かだ。  

だがWGP数は、――AMDはBoostClockでピーク性能を出すのに加え、他の要素も多くあることから単純に測ることはできないが――GameClockがNavi10 XT（RX 5700 XT）の約0.63倍なため、約1.6〜1.7倍のWGPがないとNavi10を性能で超せないことになる。  
超す必要なんてなく、モバイル向けという可能性だってあるが、VF用DeviceIDがあることと、大きく矛盾はしないが説として衝突する。  

MCM説は、正直願望によるものが大きい。  
Vega20で外部へのInfinity Fabric（以下IF）が実装され、4 GPUでのコヒーレントに対応し、Apple専用となったVega II Duoでは1つのボードにVega20を2個実装し、お互いがIFで接続されている。  
同時期に登場したRyzen 3000シリーズでは、CPU部とI/O部でダイが分けられMCM構成を取り、これもお互いにIFで接続されている。  
こうしたことでNaviでのMCM、mGPUの期待は前々からあった。  

VF用としてはVega10を2個、1つのボードに実装したV340があり、Navi12でMCMが可能とされていてもおかしくはない。  
[[3/5] drm/amdgpu: GPU TLB flush API moved to amdgpu_amdkfd](https://patchwork.freedesktop.org/patch/346176/)  
ただ、***現時点の***情報だとその可能性は断たれている。  
[drm/amd/powerplay: don't include the smu11 driver if header in smu v11 (v2) - Linux Kernel/amd-staging-drm-next](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd/powerplay/inc/smu11_driver_if_navi10.h?h=amd-staging-drm-next&id=013fd3a61a827c65491b0bb9ad3c0417f09c8146)  

 > v2: add hack for XGMI (Alex)

とあり、ここでの hack は切り刻むとか切り捨てるとかの意だと思われる。  
コードを見ても、Vega20やArcturusではあるXGMIの記述が、 smu11_driver_if_navi10.h だと消されている。  

### 要するに

まったくわからない。  
何かを肯定する材料があれば、それを否定する材料もある。  
結局は自分も矛盾を抱えたカオスしか話せない。  

こんな時はあまり考えず、別のことをしてただ待つのが一番だ。せっかくの年末、じっくり本を読んだり、映画やアニメを観たり、ゲームに熱中したり、冬の写真を撮りに行ったり、UEFIの設定を詰めたり、Firefoxのカスタムビルドに挑戦してもいいし、Mesaのビルドに慣れて最新Radeonが販売開始された時に備えるのもいい。  
わからないものに思考が支配されると変な方向に突っ走ってしまいがちだ。  
何をしていてもそのうち、また新たなパッチでNavi12の姿が少しずつ明らかになっていく。  
