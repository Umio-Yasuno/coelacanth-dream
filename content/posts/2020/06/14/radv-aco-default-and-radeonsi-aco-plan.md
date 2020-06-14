---
title: "RADV/ACO をデフォルトで有効にする動き、RadeonSIに ACO を バックエンドに用いる計画も"
date: 2020-06-14T20:22:49+09:00
draft: false
tags: [ "ACO", "RadeonSI", "RADV" ]
keywords: [ "", ]
categories: [ "Software", "AMD", "GPU" ]
noindex: false
---

AMD GPU向けのオープンソースVulkanドライバー RADV の新たなシェーダーコンパイラバックエンド、*ACO* が既存の LLVM に代わり、デフォルトで有効化される動きを見せている。  
{{< link >}}[radv: enable ACO by default (!5445) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/5445){{< /link >}}

*ACO* については、以前書いた検証記事を参照。  
{{< link >}}[RADV/ACO 検証: waifu2x-ncnn-vulakan の実行が最大3倍近く高速に | Coelacanth's Dream](/posts/2020/04/26/waifu2x-ncnn-vulkan-speedup-aco/#aco){{< /link >}}

GFX9世代(Vegaアーキテクチャ)から対応している FP16/int16 の Packed実行と、その Vulkan拡張機能のソフトウェア対応、[^1]  
一部最適化と安定性の向上により、*ACO* は LLVM同等の機能を備えることになるため、*ACO* をデフォルトで有効にする目処が付いたということだろう。  
その場合、LLVMをバックエンドに使用することはオプション扱いとなり、環境変数を指定する必要がある。  
現在の *ACO* 有効方法がそうであるため、入れ替わる形だ。  

[^1]: [WIP: radv/aco: enable FP16 features/extensions on GFX9+ (!5347) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/5347/diffs)
[^2]: [spirv,aco: set restrict by default and use SMEM if we know it's safe (!5207) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/5207)
[^3]: [aco: 8/16-bit code generation improvements (!5245) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/5245)


*ACO* により、グラフィクス処理において平均で約1.2倍の性能向上を、[^4]  
そして自分が検証した限りでは、コンピュート処理においては約3倍の性能向上を受けられる。[^5]  
*ACO* が性能に加えて機能と安定性を備え、デフォルトで有効化されるというのは素直に喜ばしいことだ。  

[^5]: [RADV/ACO 検証: waifu2x-ncnn-vulakan の実行が最大3倍近く高速に | Coelacanth's Dream](/posts/2020/04/26/waifu2x-ncnn-vulkan-speedup-aco/)
[^4]: [Radeon Software 20.10 vs. Upstream Linux AMD Radeon OpenGL / Vulkan Performance - Phoronix](https://www.phoronix.com/scan.php?page=article&item=radeon-software-20&num=6)

また、RADV の開発者の1人、[Samuel Pitoiset](https://gitlab.freedesktop.org/hakzsam)氏は、*ACO* を AMD GPU向けOpenGLドライバー、RadeonSI に移す計画はあるかという質問に対し、Yes と答えている。[^6]  

RadeonSI にも *ACO* がバックエンドに採用されれば、より多くのアプリケーションにおける処理性能が向上する。  
Linux + AMD GPUユーザーにとってかなりの朗報だ。  

[^6]: <https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/5445#note_532279>

{{< ref >}}

 * <https://xdc2019.x.org/event/5/contributions/334/attachments/431/683/ACO_XDC2019.pdf>
 * <https://archive.fosdem.org/2018/schedule/event/radeonsi/attachments/slides/2253/export/events/attachments/radeonsi/slides/2253/fosdem2018_shaders.pdf>

{{< /ref >}}
