---
title: "AMD の次世代GPU、Sienna Cichlid の Linux Kernelパッチが投稿される"
date: 2020-06-02T15:39:35+09:00
draft: false
tags: [ "Navi21", "Sienna_Cichlid", "GFX10" ]
keywords: [ "", ]
categories: [ "Hardware", "GPU", "AMD" ]
noindex: false
---

AMD の次世代GPU、*Sienna Cichlid* をサポートする一連の Linux Kernel パッチが投稿された。  
*Sienna Cichlid* GPU のメジャーバージョン(世代)は [GFX10](/tags/gfx10)、マイナーバージョンは 3 であり、  
gfx103x となる [RDNA 2](/tags/rdna_2) GPU に関連すると見られる。  
{{< link >}}[[PATCH 000/207] Add Support for Sienna Cichlid](https://lists.freedesktop.org/archives/amd-gfx/2020-June/049968.html){{< /link >}}

ディスプレイコントローラの eChipRev は `40` であり、過去に投稿されたパッチと照らし合わせると、*Navi21* のものと一致する。[^3] `Sienna Cichlid == Navi21?`

[^3]: [[PATCH 316/459] drm/amd/display: Add DCN2 and NV ASIC ID](https://lists.freedesktop.org/archives/amd-gfx/2019-June/035497.html)

VRAMには Navi1x から続いて GDDR6 を採用すると見られる。[^11]  

[^11]: [[PATCH 095/207] drm/amdgpu: add vram_info v2_5 in atomfirmware header](https://lists.freedesktop.org/archives/amd-gfx/2020-June/050059.html)

### インデックス

   * [GFX](#gfx)
   * [SDMA](#sdma)
   * [VCN 3.0](#vcn3)
   * [DCN 3](#dcn3)
   * [余談](#aside)

## GFX
Navi1x では無効化されていた 3D pipe、Async ring が *Sienna Cichlid* ではサポートされている。[^8]  
ring とは CP(Command Processor) が各ユニットへコマンドを発行する単位。  

[^8]: [[PATCH 113/207] drm/amdgpu: enable 3D pipe 1 on Sienna_Cichlid](https://lists.freedesktop.org/archives/amd-gfx/2020-June/050077.html)

Navi1x で無効化したのはユースケースがないため、という理由だったが[^9]、*Sienna Cichlid /RDNA 2* ではあるということだろうか？  
*RDNA 2* で追加される新機能にはコンソール機から、レイトレーシング、メッシュシェーダーへの対応が確認されているが、それらに関係しているのかもしれない。  

[^9]: <https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd/amdgpu/gfx_v10_0.c?h=amd-staging-drm-next&id=f091c1c70e89adca93c4f4f35dfa0abf90611453>

## SDMA
*Sienna Cichlid* は SDMAコントローラを 4基持ち、これは従来の一般向けGPUの倍の数だ。[^5]\(HPC向けの *Arcturus* は 8基)  
コントローラあたりのキュー数が減らされていないため、純粋に倍となる。[^7]  
増やした意図ははっきりせず、*Arctturus* のように XGMI/マルチGPU に最適化された SDMAコントローラという訳でもないようだ。  
追加された `atom_smc_dpm_info_v4_9` には XGMIに関するコードがあるが、*Arcturus* の v4_6 と同一であることから流用したものと思われ、*Sienna Cichlid* が実際にサポートしているかは定かでない。  

また、これまでは SDMAコントローラそれぞれのファームウェアイメージが用意され、それぞれロードするという形だったが、この部分が改良され、1つのファームウェアイメージを共有するようになった。  
*Arcturus* は末尾の番号が違う 8つのファームウェアイメージを用意することになったため、さすがに面倒、冗長だと感じたのだろうか。  

[^5]: [[PATCH 021/207] drm/amdgpu: add sdma ip block for sienna_cichlid (v5)](https://lists.freedesktop.org/archives/amd-gfx/2020-June/049985.html)
[^6]: [[PATCH 049/207] drm/amdgpu: update SDMA 5.2 microcode init](https://lists.freedesktop.org/archives/amd-gfx/2020-June/050013.html)
[^7]: [[PATCH 116/207] drm/amdkfd: Support Sienna_Cichlid KFD v4](https://lists.freedesktop.org/archives/amd-gfx/2020-June/050080.html)

## VCN 3.0 {#vcn3}
動画のデコード/エンコードを担当するマルチメディア部の VCN も更新されている。  
*Sienna Cichlid* は 2つの VCN3インスタンスを持つが、それらは非対称であり、片方がデコードのみ、もう片方がエンコードのみを担当する。[^1]  

[^1]: [[PATCH 152/207] drm/amdgpu/vcn3.0: schedule instance 0 for decode and 1 for encode](https://lists.freedesktop.org/archives/amd-gfx/2020-June/050116.html)

Unified Video Decoder という名前ながらエンコードも担当していた UVD と VCE(Video Compression Engine) が VCN(Video Core Next) に統合され、[^2]  
VCN3 ではデコード担当とエンコード担当がはっきり分かれる訳だ。  

[^2]: <https://www.x.org/wiki/RadeonFeature/#radeonuvdunifiedvideodecoderhardware>

## DCN3
DCN も更新され、DCN3 となった。  
どういった新機能があるかはまだ不明。(読み取れていない)  

一応、リソースから最大6画面出力ということは読み取れる。[^10]一般向けでミドル帯以上のGPUとして普通の規模だろう。  

[^10]: [[PATCH 197/207] drm/amd/display: Add DCN3 Resource](https://lists.freedesktop.org/archives/amd-gfx/2020-June/050165.html)

## 余談 {#aside}
最後に余談を。  
*Arcturus* ではそのコードネームの長さから略称が安定しない印象があるが、*Sienna Cichlid* はどうなるのか。  
それと、コードネームの *Cichlid (シクリッド)* は調べるまで知らなかったが、カワスズメ科の硬骨魚の総称らしい。  
分布は主にアフリカと中南米、一部が熱帯アジアであり、観賞用、食用とされている。  
*Sienna (シエナ)* には黄褐色または赤褐色の意味があり、イタリア中部の都市 Siena(シエーナ) が由来らしい。  
AMD が EPYC CPU のコードネームにイタリアの都市を用いていたことを考えると、*Sienna Cichlid* が突拍子もないコードネームという感じはしない。  
