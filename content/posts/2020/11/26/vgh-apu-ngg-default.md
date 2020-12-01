---
title: "VanGogh APU でも NGG/プリミティブシェーダーがデフォルトで有効に"
date: 2020-11-26T05:02:03+09:00
draft: false
tags: [ "VanGogh", "RDNA_2" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
toc: false
---

オープンソースドライバーである **RadeonSI (OpenGL)** 、**RADV (Vulkan)** は、*RDNA /GFX10* 世代からの GPU に実装されている *NGG (Next Generation Geometry)/プリミティブシェーダー* をサポートし、一部を除いたほとんどはデフォルトで有効にされる。  
*RDNA 2 / GFX10.3* 世代の GPU を持つ *VanGogh APU* はこれまでその一部に含まれ、専用 VRAM を持つかどうかの判定で弾かれてドライバー部分では無効化されていたが、[^vgh-disable-ngg]  
その判定を取り払い、*VanGogh APU* でもデフォルトで *NGG* を有効にするパッチが投稿されている。この変更は **RadeonSI (OpenGL)** 、**RADV (Vulkan)** 両方に適用されている。  
{{< link >}} [radv: enable NGG on GFX10.3 APUs by default (9f0133d9) · Commits · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/commit/9f0133d961fe44f3057821b596c58c71557ab595?merge_request_iid=7769) {{< /link >}}
{{< link >}} [radeonsi: tiny miscellaneous changes, NGG enabled on VanGogh, 1 optimization for dead PS outputs (!7721) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/7721/diffs?commit_id=de799b2270f5342c2c108488c2c694412b06c945) {{< /link >}}

[^vgh-disable-ngg]: [新たな AMD RDNA 2 GPU、"Dimgrey Cavefish" & "VanGogh" | Coelacanth's Dream](/posts/2020/09/23/amd-vangogh-dimgrey_cavefish/)

*NGG* については以下。  
{{< link >}} [ACOバックエンドでも NGG をサポートする動き | Coelacanth's Dream](/posts/2020/10/04/aco-ngg-gfx10/) {{< /link >}}
{{< link >}} [RadeonSIドライバー + RDNA 2 では NGGカリング/プリミティブシェーダー がデフォルトで有効に | Coelacanth's Dream](/posts/2020/10/17/gfx103-default-ngg-culling/) {{< /link >}}
詳細はそれら記事に書いてあるが、*NGG* を有効にすることで、これまでのグラフィクスパイプラインは必要だった、ジオメトリシェーダーの結果を一度 VRAM に出力する手間を減らすことができる。  
dGPU と比べてメモリ帯域がずっと狭い APU では、このことによる性能向上への効果がより期待できるのではないかと思う。  
*NGGカリング* によってカリング処理の効率も大きく引き上げられる。  

それと、それら記事を書いた時は *NGGカリング == プリミティブシェーダー* のように書いたが、  
*NGG == プリミティブシェーダー* 、*NGGカリング == プリミティブカリング* という対応関係が正しいようだ。  
*NGG* を他のシェーダーステージに合わせた呼び方が *プリミティブシェーダー* で、略称としては、*Primitive Shader* をそのまま *PS* としてしまうと *Pixel Shader* と被り、大変ややこしくなるため、*NGG* を略として用いる、ということと思われる。  

 >           NGG, /* Primitive shader, used to implement VS, TES, GS. */
 > {{< quote >}} [src/amd/compiler/aco_ir.h · 34bc9477de18a92e76ea7c536940a631323a83b6 · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/blob/34bc9477de18a92e76ea7c536940a631323a83b6/src/amd/compiler/aco_ir.h#L1537) {{< /quote >}}

## 魅力の多い VanGogh APU

*VanGogh* は LPDDR4/xメモリよりも高速かつ省電力な LPDDR5メモリをサポートするとされている。*VanGogh* が LPDDR5-5500 128-bit をサポートすると *仮定* して、*Renoir* と比較すると、ピークメモリ帯域は約 1.3倍となる。(88.0 GB/s vs 68.3 GB/s)  
{{< link >}} [Linux Kernel に AMD の次世代 APU "VanGogh" をサポートするパッチが投稿される | Coelacanth's Dream](/posts/2020/09/26/amd-vgh-linux-kernel-patch/) {{< /link >}}
また、*VanGogh* は *Vega /GFX9* 世代以降の GPU を搭載する {{< comple >}} Xbox Series X|S、PS5 のゲーム機向け APU を除けば  {{< /comple >}} 初の APU である。 APU は *Raven* からしばらく、GPU の世代は *Vega /GFX9* から進まなかった。  
*RDNA 2 /GFX10.3* 世代となることで、グラフィクス性能だけでなく、混合精度関連の命令サポートによるコンピュート性能の向上も期待できる。  

そのため、メモリの高速化、GPU部に *RDNA 2 /GFX10.3* 世代の採用、*NGG* サポートによる GPU性能の向上、それと初の *RDNA系アーキテクチャ* 採用 APU としてその構成等、*VanGogh* には期待点、未知の部分が多く {{< comple >}} これらはそのまま魅力とも言い換えられる {{< /comple >}}、個人的に *VanGogh* は特に正式発表が待ち遠しい APU である。  
