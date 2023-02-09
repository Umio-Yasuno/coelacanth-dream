---
title: "AMD Phoenix と Phoenix2 の CPUID, GFX1103_R1 と GFX1103_R2"
date: 2023-02-07T00:28:57+09:00
draft: false
categories: [ "Hardware", "AMD", "APU" ]
tags: [ "Phoenix", "Phoenix2", "RDNA_3", "GFX11", "Coreboot", "CPUID" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

## Phoenix と Phoenix2 の CPUID Family/Model/Stepping {#cpuid}

Felix Held 氏により、*AMD Phoenix APU* と *AMD Phoenix2 APU* の `CPUID Family/Model/Stepping` に関するパッチが Coreboot に投稿されている。  
Coreboot はオープンソースなファームウェア、BIOS/UEFI を開発するプロジェクトであり、Felix Held 氏は現在フルタイムで Coreboot プロジェクトの開発に参加している。  

 * [soc/amd/phoenix/include/cpu: rename CPUID define to match CPU model (Ie7500130) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/72843/1)
 * [soc/amd/phoenix/include/cpu: add Phoenix CPUID (Id4eb3502) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/72853/1/)

 >         #define PHOENIX_A0_CPUID	0x00a70f40
 >         #define PHOENIX2_A0_CPUID	0x00a70f80
 >
 > {{< quote >}} <https://review.coreboot.org/c/coreboot/+/72843/1/src/soc/amd/phoenix/include/soc/cpu.h#b6> {{< /quote >}}

Felix Held 氏のコメントによれば、`0x00a70f80 (Family: 0x19, Model: 0xA8, Stepping: 0)` は *Phoenix2* の `CPUID Family/Model/Stepping` であり、*Phoenix APU* は `0x00a70f40 (Family: 0x19, Model: 0xA4, Stepping: 0)` となる。  
前回 Coreboot で *Phoenix APU* のサポートが進められていることを取り上げた時には、エンジニアサンプリング品やリビジョンによって `CPUID Model` を変えていると考えたが、少なくとも *Phoenix APU* と *Phoenix2 APU* はそれに関係なく明確に違う `CPUID Model` が割り当てられているようだ。[^phx-coreboot]  

[^phx-coreboot]: [Coreboot で Phoenix APU のサポートが進められる | Coelacanth's Dream](/posts/2023/01/11/coreboot-phoenix/)

`CPUID` は実行した CPU の情報を返す命令であり、EAX/ECX レジスタの値によって情報の種類は変わる。  
`CPUID Family/Model/Stepping` は EAX レジスタの値が `0x1` の時に返される情報であり、ソフトウェアはそれを用いてプロセッサの種類を判定することができる。  

*Phoenix* と *Phoenix2* の関係性はコードネーム的にも *Raven* と *Raven2 (Dali/Pollock)* に近いものと思われる。  

## GFX1103_R1, GFX1103_R2 {#gfx1103_r1_r2}
AMD のソフトウェア開発者である Marek Olšák 氏により、*Phoenix APU* の GPU ID (GFX ID) となる *GFX1103* を *GFX1103_R1* と *GFX1103_R2* に分けるパッチが Mesa3D に投稿され、それを含めたマージリクエストは既に main ブランチに取り込まれている。  

 * [amd: split GFX1103 into GFX1103_R1 and GFX1103_R2 (84d59cdb) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/84d59cdb5971424a4297e288b852c8cc15c46163)
 * [amd: update the cache size for gfx1103_r1 (76e3437c) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/76e3437c1ed88bb63c64ff87654224aee4ab0091)

*GFX1103_R1* と *GFX1103_R2* の違いとしては、L2キャッシュブロックあたりのサイズが明らかにされており、*GFX1103_R1* が *Rembrandt (Yellow Carp)* と同じ 512KiB なのに対し、*GFX1103_R2* は 256KiB となっている。  
ここでは *GFX1103_R2* を指定したコードは追加されていないが、`switch` 文の `default` ケースが 256KiB となっている。  
また、AMD GPU は基本 L2キャッシュブロックをメモリチャネルと同数持ち、*Phoenix APU (GC 11.0.1)* は合計 L2キャッシュサイズ 2MiB を持つことが過去に amd-gfx メーリングリストにて明かされている。[^phx-cache]  

[^phx-cache]: [L0ベクタキャッシュと GL1データキャッシュが増やされる RDNA 3/GFX11 APU | Coelacanth's Dream](/posts/2022/09/02/gfx11-l0c-gl1c/)

 >             case CHIP_REMBRANDT:
 >         +   case CHIP_GFX1103_R1:
 >                info->l2_cache_size = info->num_tcc_blocks * 512 * 1024;
 >                break;
 >             }
 >
 > {{< quote >}} [amd: update the cache size for gfx1103_r1 (76e3437c) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/76e3437c1ed88bb63c64ff87654224aee4ab0091) {{< /quote >}}

他では、`chip_rev` が `0` の場合、シェーダープリフェッチを無効化する対象に *GFX1103_R1* は含まれるが、*GFX1103_R2* は含まれていない。  
しかし、Marek Olšák 氏による他のパッチにて当該部分のコードは削除されているため、あまり気にする必要のない情報ではある。そのパッチによれば、シェーダープリフェッチ機能の無効有効はハードウェア側で自動で行われ、ソフトウェア側で一部のチップに存在するバグに対する回避策は必要としない。  

 * [radeonsi/gfx11: remove the INST_PREF_SIZE workaround (d21850f7) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/d21850f7538cbf719792e74cbc78b3c638b26137)

もう 1つの *RDNA 3 APU* という点では、以前 AMDGPU ドライバーにサポートを追加するパッチが投稿された *GC 11.0.4 APU* が思い出される。[^gc_11_0_4]  
ただ、*GFX1103_R1* と *GFX1103_R2* を判定する部分では `eRivisionID, external_rev_id` が用いられているが、AMDGPU ドライバーのコードを見るに、*Phoenix APU (GC 11.0.1)* と *GC 11.0.4* の `rev_id` に追加されるオフセット値は同じであり、これまでのパターンから `eRivisionID, external_rev_id` も同一である可能性が高い。  
そうなると、*Phoniex APU (GC 11.0.1)* も *GC 11.0.4* も *GFX1103_R1* として判定される。  
現時点で *Phoenix APU* として *GC 11.0.1, GFX1103_R1*、*GC 11.0.4, GFX1103_R1*、*GC 11.0.x?, GFX1103_R2* の 3種類が存在することが考えられるが、前者 2つに関しては単にステッピングの違いを表現しただけの可能性もある。  
Coreboot にて存在が明かされた *Phoenix2* も、GC IPバージョンや DeviceID が不明なため、*GFX1103_R1* と *GFX1103_R2* のどちらが採用されているかは不明である。  

 * [[PATCH] drm/amdgpu: add soc21 common ip block support for GC 11.0.1](https://lists.freedesktop.org/archives/amd-gfx/2022-May/078678.html)
 * [[PATCH 09/19] drm/amdgpu: add soc21 common ip block support for GC 11.0.4](https://lists.freedesktop.org/archives/amd-gfx/2022-November/086816.html)

[^gc_11_0_4]: [もう 1つの RDNA 3 APU IP、GC 11.0.4 | Coelacanth's Dream](/posts/2022/11/22/rdna_3-apu-gc_11_0_4/)
