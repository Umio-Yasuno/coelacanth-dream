---
title: "AMDの次世代 RDNA 2 GPU、\"Navy Flounder\" のLinux Kernelパッチが投稿される"
date: 2020-07-15T03:53:22+09:00
draft: false
tags: [ "RDNA_2", "Navi22", "Navy_Flounder" ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
---

AMDの次世代 [RDNA 2](/tags/rdna_2) GPU、*Navy Flounder* をサポートする最初のLinux Kernelパッチ一連が投稿された。  
{{< link >}} [[PATCH 00/42] Navy Flounder support](https://lists.freedesktop.org/archives/amd-gfx/2020-July/051526.html) {{< /link >}}

*Navy Flounder* は [Sienna Cichlid](/tags/sienna_cichlid)と同じ世代(GFX10)、同じメジャーバージョンである GFX10.3 となっている。  
マルチメディア部(VCN3)、ディスプレイコントローラ(DCN3)、SDMAコントローラのIPバージョンもまた *Sienna Cichlid* と共通するが、VCN3 と SDMAコントローラの規模の点で異なり、*Navy Flounder* の方が小さい。  

*Sienna Cichlid* はディスプレイコントローラの eChipRev から *Navi21* と考えられたが、  
{{< link >}} [AMD の次世代GPU、Sienna Cichlid の Linux Kernelパッチが投稿される | Coelacanth's Dream](/posts/2020/06/02/amd-sienna_cichlid/) {{< /link >}}
*Navy Flounder* は external_rev_id が ` + 0x32` となっており[^navy-flounder-e_rev_id]、過去に誤って投稿されたであろう *Navi22* の範囲と一致することから[^navi22-e_rev_id]、`Navy Flounder == Navi22` と考えられる。  

[^navy-flounder-e_rev_id]: [[PATCH 09/42] drm/amdgpu/soc15: add support for navy_flounder](https://lists.freedesktop.org/archives/amd-gfx/2020-July/051535.html)
[^navi22-e_rev_id]: [pal/amdgpu_asic_addr.h at 39abe2297ca58a2b84dcd9bc5e238fbc399bd6e0 · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/39abe2297ca58a2b84dcd9bc5e238fbc399bd6e0/src/core/imported/addrlib/src/amdgpu_asic_addr.h#L111)

## Sienna Cichlid と規模が異なる Navy Flounder
上述したように、*Navy Flounder* は *Sienna Cichlid* と各IPが共通しており、VCN3 と SDMAコントローラの規模が異なっている。順当に考えればこれは GPU としての規模、シェーダープロセッサや RenderBackend、メモリバス幅の規模を表しているように考えられる。  

また、*Sienna Cichlid* では GFX Pipe に Async ring のサポートが追加されていたが[^sienna_cichlid-async-ring]、後のパッチにて現時点ではそれを無効化し、Primary ring のみとしている。[^sienna_cichlid-one-pipe]  

[^sienna_cichlid-async-ring]: [[PATCH 113/207] drm/amdgpu: enable 3D pipe 1 on Sienna_Cichlid](https://lists.freedesktop.org/archives/amd-gfx/2020-June/050077.html)
[^sienna_cichlid-one-pipe]: [[PATCH 162/207] drm/amdgpu: only use one gfx pipe for Sienna_Cichlid](https://lists.freedesktop.org/archives/amd-gfx/2020-June/050126.html)

そして *Navy Flounder* は GFX Pipe のコードにて分岐先を *Sienna Cichlid* と共通しているため[^navy_flounder-switch]、*Navy Flounder* もハードとしては Async ring をサポートしているものと思われる。  

[^navy_flounder-switch]: [[PATCH 15/42] drm/amdgpu: add gfx ip block for navy_flounder](https://lists.freedesktop.org/archives/amd-gfx/2020-July/051541.html)

### 1インスタンスの VCN3?
まず、*Sienna Cichlid* は非対称の VCN3 を 2インスタンス持ち、片方がデコードを、片方がエンコードのみを担当する形となっていた。[^sienna_cichlid-vcn]  
しかし今回のパッチでは、*Navy Flounder* は 1インスタンスだけ持つとされている。[^navy_flounder_vcn_1]  
ただ、同時にパッチ内のコメントで PG(Power Gating, 使用していない、アイドル状態のユニットの電源を遮断し、リーク電流を削減する省電力技術) を設定する上では 2インスタンス(VCN0/VCN1)に分かれていた方が都合が良く、電力効率に優れるとあるため、  
VCN3 が内部的に 2インスタンスとなっているのか、それともやはり *Sienna Cichlid* は 2インスタンス持ち、*Navy Flounder* は 1インスタンス持つ構成になっているのか、イマイチはっきりしない。  
各IPが *Sienna Cichlid* と共通するためか、*Navy Flounder* の一連のパッチは *Sienna Cichlid* ほど巨大ではなく、VCN3 のインスタンスがどうなっているかを示すコードは今回追加されていなかった。  

{{< ins >}}

VCN3.0部のコードを見るに、2インスタンス持つ場合に実行する制御文の式が、*Sienna Cichlid* かどうかで判定されており、それとそれ以降の GPU という書き方ではないため、  
VCN3.0 を、*Sienna Cichlid* は 2インスタンス、*Navy Flounder* は 1インスタンスを持つようになっていると思われる。  
{{< link >}} <https://cgit.freedesktop.org/~agd5f/linux/tree/drivers/gpu/drm/amd/amdgpu/vcn_v3_0.c?h=amd-staging-drm-next-navy-flounder&id=d5ba938e1cdda54b4fb2ca2c676a77b4d887f055> {{< /link >}}

{{< /ins >}}

[^sienna_cichlid-vcn]: [[PATCH 152/207] drm/amdgpu/vcn3.0: schedule instance 0 for decode and 1 for encode](https://lists.freedesktop.org/archives/amd-gfx/2020-June/050116.html)
[^navy_flounder_vcn_1]: [[PATCH 38/42] drm/amd/powerplay: set VCN1 pg only for sienna_cichlid](https://lists.freedesktop.org/archives/amd-gfx/2020-July/051564.html)

### Sienna Cichlid の半分となる、標準的な SDMAコントローラ数
*Sienna Cichlid* は具体的な理由は明かされず、コンシューマ向けAMDGPUとしては多い 4基の SDMAコントローラを持つが、*Navy Flounder* は一転、標準的な 2基のSDMAコントローラを持つ。[^navy_flounder_sdma]  
このことで SDMAコントローラを 4基持つのは *GFX10.3/ RDNA 2* で共通した構成ではなく、*Sienna Cichlid* 特有という意味が強まった。  
*Sienna Cichlid* がそれ程に持つ理由があるはずだが、まだ想像するしかない段階にある。  
*Arcturus* のように マルチGPU(XGMI) に最適化されてもいないため、単純にマルチGPUのためとは言えないように思う。  

## 色+魚 な RDNA 2 のコードネーム
*Sienna Cichlid* というコードネームを分解すると、黄褐色または赤褐色を意味する *Sienna (シエナ)* とカワスズメ科の硬骨魚 *Cichlid (シクリッド)* に分かれるが、  
*Navy Flounder* は、濃紺色を意味する *Navy (ネイビー)* 、砂底に生息する硬骨魚 *Flounder (フラウンダー) [カレイ]* に分かれる。  
ローマの都市、画家と続いて、次は色と硬骨魚がコードネームに使われるようだ。  

ちなみに *Cichlid (シクリッド)* も *Flounder (フラウンダー) [カレイ]* も食用。  

[^navy_flounder_sdma]: [[PATCH 16/42] drm/amdgpu: add sdma ip block for navy_flounder](https://lists.freedesktop.org/archives/amd-gfx/2020-July/051542.html)
