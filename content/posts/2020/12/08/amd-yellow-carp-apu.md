---
title: "新たな AMD APU、\"Yellow Carp\"　―― SMU は v13 に"
date: 2020-12-08T15:26:55+09:00
draft: false
tags: [ "Yellow_Carp", ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
toc: false
---

Linux Kernel (amd-gfx) に、新たな APU *Yellow Carp* の SMU (System Management Unit) v13 をサポートするパッチが投稿された。  
{{< link >}} [[PATCH 5/8] drm/amd/pm: add smu13 ip support for moment(V3)](https://lists.freedesktop.org/archives/amd-gfx/2020-December/057162.html) {{< /link >}}
{{< link >}} [[PATCH 6/8] drm/amd/pm: add yellow_carp_ppt implementation(V3)](https://lists.freedesktop.org/archives/amd-gfx/2020-December/057163.html) {{< /link >}}

タイトルから今回の SMU v13 をサポートするパッチは部分的なものであり、近く *Yellow Carp* をサポートする一連のパッチが投稿されるものと思われる。  
今回のパッチから得られる情報は少ないが、*Yellow Carp* が APU であることは以下の引用部分から読み取れる。  

 >        +void yellow_carp_set_ppt_funcs(struct smu_context *smu)
 >        +{
 >        +	smu->ppt_funcs = &yellow_carp_ppt_funcs;
 >        +	smu->message_map = yellow_carp_message_map;
 >        +	smu->table_map = yellow_carp_table_map;
 >        +	smu->is_apu = true;
 >        +}
 >
 > {{< quote >}} [[PATCH 6/8] drm/amd/pm: add yellow_carp_ppt implementation(V3)](https://lists.freedesktop.org/archives/amd-gfx/2020-December/057163.html) {{< /quote >}}

SMU v13 と括られてはいるが、追加されたファイルやパッチ内のメッセージを読むと、*Yellow Carp* の SMU は v13.0.1 というバージョンとなっている。  
推察するに、初期の SMU v13 IP には問題があり、それを修正したため、僅かにバージョンを上げたのかもしれない。  
GPUコア部の世代等はまだ不明。  

SMU は最近の AMD GPU/APU に限って言えば、v11 と v12 に分かれており、  
*RDNA / GFX10* 世代の dGPU、[Navi10](/tags/navi10) / [Navi12](/tags/navi12) / [Navi14](/tags/navi14)、*RDNA 2 / GFX10.3* 世代の dGPU/APU、[Sienna Cichlid](/tags/sienna_cichlid)、[Navy Flounder](/tags/navy_flounder)、[Dimgrey Cavefish](/tags/dimgrey_cavefish)、[VanGogh](/tags/vangogh)、それと [Arcturus](/tags/arcturus) は SMU v11、  
[Renoir](/tags/renoir) は SMU v12 となる。[Green Sardine](/tags/green_sardine) は不明だが、恐らく Renoir と同じ SMU v12 と思われる。  

コードネームについては、{{< comple >}} VanGogh は外れているが {{< /comple >}} [Sienna Cichlid](/tags/sienna_cichlid) から始まる X11 color + 魚 を組み合わせたものとなっている。  
<span style="color:yellow">Yellow はここの文字の色。 Carp は日本だと馴染みのあるコイの英名。</span>


