---
title: "2つの非同期パイプラインが追加される Xe-HP EU"
date: 2021-04-02T16:24:19+09:00
draft: false
tags: [ "Xe-HP", ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "GPU" ]
noindex: false
# summary: ""
---

オープンソースで開発される Intel GPU のシェーダーコンパイラバックエンドに、*{{< xe class="hp" >}}* GPU の ISA (Instruction Set Architecture) をサポートするパッチ (マージリクエスト) が投稿された。  
偶然だが、そのマージリクエストが Mesa3D の Gitlabレポジトリにおける 1000個目のマージリクエストであり、次世代の Intel GPU をサポートするパッチは記念としてぴったりと言えるかもしれない。  

 * [intel: Implement compiler support for XeHP graphics platforms. (!10000) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/10000)

*{{< xe class="hp" >}}* は、*{{< xe class="lp" >}}* から対応していた `DP4A` 命令に加え、BFloat16フォーマットや倍精度演算 (DP) に対応した機械学習に最適化されたアーキテクチャである。  

## {{< xe class="lp" >}} から拡張される {{< xe class="hp" >}} EU {#xehp-eu}

パッチを投稿した Intel の [Francisco Jerez](https://gitlab.freedesktop.org/currojerez) 氏によるコメントでは、*{{< xe class="hp" >}}* EU は複数の非同期ALUパイプラインを持ち、  
それら複数のパイプラインは、既存の FPUパイプライン {{< comple >}} *Tiger Lake*, *Rocket Lake*, *Alder Lake*, *DG1* などに採用されている {{< xe class="lp" >}} EU のことを指していると思われる {{< /comple >}} から複数の非同期パイプラインに分割したとも説明している。  
パイプラインは浮動小数点 (FP)、整数 (Int)、倍精度浮動小数点 (DP) に分けられる。  

また、パイプライン間のデータ依存関係についてはコンパイラ側で制御、管理するとし、そのためのパッチも含まれている。  

 > This MR implements compiler support for the ISA of the Intel XeHP family of GPUs. The most invasive compiler changes relative to previous generations are the result of the preexisting FPU pipeline being split into multiple asynchronous pipelines (a floating-point, integer and long AKA double-precision pipeline), which is highly visible to software because the hardware is not able to guarantee data coherency across instructions (already since TGL), so the compiler is now responsible for keeping track of which pipeline will be executing which instruction and specifying synchronization primitives in order to resolve any cross-pipeline dependencies.
 >
 > {{< quote >}} [intel: Implement compiler support for XeHP graphics platforms. (!10000) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/10000/) {{< /quote >}}
 >
 >        The execution units of XeHP platforms have multiple asynchronous ALU
 >        pipelines instead of (as far as software is concerned) the single
 >        in-order pipeline that handled most ALU instructions except for
 >        extended math in the original Xe.  It's now the compiler's
 >        responsibility to identify cross-pipeline dependencies and insert
 >        synchronization annotations whenever necessary, which are encoded as
 >        some additional bits of the SWSB instruction field.
 >
 > {{< quote >}} [intel: Implement compiler support for XeHP graphics platforms. (!10000) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/10000/diffs?commit_id=53af9902f1d9748fe4856e08203a7e4d2129ea72) {{< /quote >}}

*{{< xe class="lp" >}}* EU (Execution Unit) は以下のように、命令を発行する Issue Port 0 は FP/INT演算器を、Port 1 は三角関数等の超越関数を実行する EM (Extended Math)演算器を担当する構成となっている。  
[Francisco Jerez](https://gitlab.freedesktop.org/currojerez) 氏のコメントと合わせれば、 *{{< xe class="lp" >}}* EU では、EM演算器で処理される演算命令を除いて、FP/INT が同一のパイプラインでインオーダー方式に処理するアーキテクチャだったと説明できる。  

{{< figure src="/image/2021/04/02/xe-lp-eu.webp" title="Xe-LP Execution Unit" caption="画像出典: [HotChips2020_GPU_Intel_Xe_David_Blythe.pdf](https://www.hotchips.org/assets/program/conference/day1/HotChips2020_GPU_Intel_Xe_David_Blythe.pdf)" >}}

コード内のコメントでは、(*{{< xe class="lp" >}}* から) 2つの非同期ALUパイプラインが追加されているとしており、上述の説明と合わせるに、*{{< xe class="lp" >}}* では FP/INT演算器に接続されていた Issue Port 0 を分割、それぞれに独立した Issue Port を搭載し、そして倍精度演算器と一緒に Issue Port をもう 1基搭載したのが *{{< xe class="hp" >}}* EU の構成ではないかと考えられる。  
Issue Port が増えればその分多く演算命令を発行でき、多くの処理を行うことができる。また非同期に演算処理を行うことも可能になる。  
こうしたアーキテクチャの変更は、FP32 と INT32 の演算器を並列に動作できるにした NVIDIA Turing GPU のそれと近いように思える。[^turing]  
コンパイラ側で制御する必要はあるが、パッチの日付は 1,2年前のものとなっており、だいぶ以前から取り組んでいたことが窺える。*{{< xe class="hp" >}}* について内部で開発していたものを、サポートのためオープンにする段階に来たとも言える。  
*{{< xe class="hp" >}}* の特徴として以前より IPC の向上がアピールされていたが、そのためのアーキテクチャ改良の 1つが Issue Port の分割、増設なのだろう。[^xehp-ipc]  

[^xehp-ipc]: [Intel’s Xe GPUs — from Laptops to Supercomputers | EE Times](https://www.eetimes.com/intels-xe-gpus-from-laptops-to-supercomputers/2/)
[^turing]: [NVIDIAのレイトレーシングGPU「Turing」を読み解く - Hot Chips 31 (1) | TECH+](https://news.mynavi.jp/article/20191024-912855/)

 >        /**
 >         * TGL+ SWSB RegDist synchronization pipeline.
 >         *
 >         * On TGL all instructions that use the RegDist synchronization mechanism are
 >         * considered to be executed as a single in-order pipeline, therefore only the
 >         * TGL_PIPE_FLOAT pipeline is applicable.  On XeHP+ platforms there are two
 >         * additional asynchronous ALU pipelines (which still execute instructions
 >         * in-order and use the RegDist synchronization mechanism).  TGL_PIPE_NONE
 >         * doesn't provide any RegDist pipeline synchronization information and allows
 >         * the hardware to infer the pipeline based on the source types of the
 >         * instruction.  TGL_PIPE_ALL can be used when synchronization with all ALU
 >         * pipelines is intended.
 >         */
 >
 > {{< quote >}} [intel: Implement compiler support for XeHP graphics platforms. (!10000) · Merge Requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/10000/diffs?commit_id=53af9902f1d9748fe4856e08203a7e4d2129ea72#diff-content-aa711c4d6f41feb214c7f983adc5e0e3e95740e0) {{< /quote >}}

| {{< xe >}} EU | IP 0 | IP 1 | IP 2 | IP 3 |
| :-- | :--: | :--: | :--: | :--: |
| {{< xe class="lp" >}} | FP/INT | EM | - | - |
| {{< xe class="hp" >}} | INT?<br>(or FP?) | EM | FP?<br>(or INT?) | DP? |

*{{< xe class="hp"  >}}* は、タイルと呼ぶ GPUチップを、パッケージ上に単体もしくは複数 (2, 4) 搭載する構成を取り、1タイルあたりに EU を 512基搭載している。  
Intel Architecture Day 2020 で披露されたデモでは、1タイル (512EU) 構成の *{{< xe class="hp" >}}* が 1300MHz で動作し、約10.6 TFLOPs というピークFP32演算性能を達成していた。  
*{{< xe class="lp" >}}* の同様の 8-wide FPパイプライン 1ポートという構成では、ピークFP32演算性能は約5.3TFLOPs と一致しなかったが、*{{< xe class="hp" >}}* で増設された倍精度演算器のパイプラインも単精度演算器として活用していたとすると約10.6TFLOPsというデモの性能と近い数字が出る。  
パイプラインあたりの演算幅を増やした可能性もあるが、今回のパッチで明かされた情報と合わせて上のような推測もできる。  
{{< link >}} [Intel Architecture Day 2020 個人的まとめ　―― XeHP は 1-Tile 512EU、XeLPアーキテクチャ詳細 | Coelacanth's Dream](/posts/2020/08/14/intel-architecture-day-2020/#xe-hp-1t-512eu) {{< /link >}}
ゲーミング向け {{< xe >}} アーキテクチャ *{{< xe class="hpg" >}}* の存在が明らかにされているが、 *{{< xe class="lp" >}}* と *{{< xe class="hp" >}}* どちらの EU構成を踏襲するかはまだ分かっていない。  

パッチでは他に、ハードウェアから整数除算演算器が外されたことや、倍精度演算対応のためと思われる 64-bitレジスタについても触れられている。  

{{< ref >}}
 * [Intel’s Xe GPUs — from Laptops to Supercomputers | EE Times](https://www.eetimes.com/intels-xe-gpus-from-laptops-to-supercomputers/2/)
 * [HotChips2020_GPU_Intel_Xe_David_Blythe.pdf](https://www.hotchips.org/assets/program/conference/day1/HotChips2020_GPU_Intel_Xe_David_Blythe.pdf)
 * [A case for a faster and simpler driverNIR on the Mesa i965 backend](https://archive.fosdem.org/2016/schedule/event/i965_nir/attachments/slides/1113/export/events/attachments/i965_nir/slides/1113/nir_vec4_i965_fosdem_2016_rc1.pdf)
 * <https://andosprocinfo.web.fc2.com/Myweb/wadai20/20201219.htm>
{{< /ref >}}
