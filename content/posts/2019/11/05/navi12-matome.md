---
title: "Navi12 Matome"
date: 2019-11-05T15:30:17+09:00
draft: false
tags: [ "Radeon", "RDNA", "Navi12", "GFX10", "gfx1011" ]
keywords: [ "Radeon", "Navi12" ]
categories: [ "Hardware", "AMD", "GPU" ]
---

まだ製品未発表のNavi12についてのまとめ。  
噂や推測が多分に含まれます。  

### 性能面
今の所ほとんど噂。  
(Navi10より?)ビックダイになるとか、メモリバス幅は256-bitのままとか言われてる。  
後者のメモリバス幅は一応根拠があり、  
[ ac: move num_sdp_interfaces into radeon_info ](https://gitlab.freedesktop.org/mesa/mesa/commit/deab3a23f6c35720248144637058697f46b2fa34)  
このコミットにて num_sdp_interfaces の数がNavi10と同じ、=256-bitだと言われている。  
後にここの部分のコードは、( num_sdp_interfaces = L2キャッシュのブロック数 )となり、L2キャッシュはメモリコントローラーに接続されていることから、256-bitというのは確度が高いと思われる。  
そうそう設計を変更するとも思いにくい。  
それとSDPというのは Scalable Data Port の略で、SDP自体はI/O、マルチメディア(VCE、UVD、VCN等)、ディスプレイ（DCE、DCN）、メモリコントローラーとの接続に使われる。  

メモリバス幅を増やさないことの理由も大体想像できる。  
微細化によってコア部分が小さくなっていくのに対して、メモリコントローラー部分はそれほど小さくならない。  
Zen2ではそのことからメモリコントローラーを12nmのcIODへと分離した。  
メモリコントローラーを外周部に配置しなければいけないことから、ダイサイズを概ね決めてしまうというのもある。  
要はメモリバス幅を増やすというのはコスト的に効率が良い訳ではなく、7nmでのコストの高さから下手にダイサイズを大きくすることは避けると思われる。  
長くなるので多くは語らないが、Navi10はメモリ帯域に対してFP32ピーク性能がそこまで高くなく、RDNAではL1キャッシュが追加されたのもあって、帯域としてはWGPを増やす余地があるはずだ。  
  
### その他
[drm/amdkfd: Add NAVI12 support from kfd side](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd/amdkfd?h=amd-staging-drm-next&id=93de2d818be3b3c880713fe682f8519c42ac2080)  
Navi12にはVF(Virtual Function)用のDeviceIDが用意されており、  
VFは従来製品で言えば、MI6(Polaris10 36CU 16GB)、MI25(Vega10 64CU 16GB)、V340(Vega10 2x56CU 32GB)が担ってきた。  
このこともNavi12がNavi10よりも上の性能ということを裏付けしていると言えるだろう。  
VFではメモリ容量16GB以上が求められる。  
Navi12でメモリコントローラーの数は増えない（と見ている）が、それでも容量を増やすことはできる。  
GDDR6チップとコントローラーをそれぞれ16-bitで接続することで、バス幅はそのままにチップ数（容量）を倍にする方法と、  
16Gbチップを採用する方法がある。  
どっちかでも16GBに達するが、どうせやるなら両方をとって最大32GBにするように思う。  

