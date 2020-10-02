---
title: "AMD Renoir の兄弟？ Linux Kernel に \"Green Sardine\" APU をサポートするパッチが投稿される "
date: 2020-10-03T00:17:14+09:00
draft: false
tags: [ "Green_Sardine", "Renoir", "Lucienne", "Cezanne" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
toc: false
---

Linux Kernel(amd-gfx) に *Green Sardine* APU をサポートする最初のパッチが投稿された。  
{{< link >}} [[PATCH 1/7] drm/amdgpu: add Green_Sardine APU flag](https://lists.freedesktop.org/archives/amd-gfx/2020-October/054441.html) {{< /link >}}

*Green Sardine* の詳細はまだ不明だが、追加されたパッチでは *Renoir* の部分に *Green Sardine* かの判定が追加されており、GPU部としては *Renoir* と変わらないものと思われる。   
パッチの規模がそう大きくないのも、*Renoir* のコードをそのまま使えるからだろう。  

 >           case CHIP_RENOIR:
 >       -      chip_name = "renoir";
 >       +      if (adev->apu_flags & AMD_APU_IS_RENOIR)
 >       +         chip_name = "renoir";
 >       +      else
 >       +         chip_name = "green_sardine";
 >              break;
 >
 > {{< quote >}} [[PATCH 2/7] drm/amdgpu: add green_sardine support for gpu_info and ip block setting (v2)](https://lists.freedesktop.org/archives/amd-gfx/2020-October/054442.html) {{< /quote >}}

そして、GPU部が *Renoir* という点では、現在 2つの未発表の APU が確認されており、  
以前より Coreboot 等で名前が出てきていた、CPU/GPUアーキテクチャは *Renoir* から変わらないとされる [Lucienne](/tags/lucienne) APU と、  
CPUアーキテクチャは *Zen 3* になると噂されている [Cezanne](/tags/cezanne) APU のどちらかが *Green Sardine* ではないかと考えられる。  

ただ、*Lucienne* は、[Raven](/tags/raven) に対する *Picasso* のように GFXコア、SDMA、ディスプレイコントローラー、マルチメディアエンジン等の各IPが変わらないと考えられるが、*Cezanne* は GFXコア部以外は *Renoir* からバージョンアップされる可能性がある。  
そのため、どちらかと言えば *Lucienne* が *Green Sardine* であるように思うが、これ以上は AMD が投稿するパッチを待つ他ない。  

そして、*Ravenファミリー* はまだまだややこしくなるようだ。  


ちなみに *Sardine* はイワシの英名。  
