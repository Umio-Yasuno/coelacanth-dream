---
title: "Ryzen 4000 U/H-Series (Renoir) 考察"
date: 2020-01-07T21:25:17+09:00
draft: true
draft: false
tags: ["Radeon", "GCN", "Raven", "GFX9", "gfx909", "Renoir", "Zen2" ]
keywords: [ "Radeon", "Renoir", "APU" ]
categories: [ "Hardware", "AMD", "APU", "GPU" ]
---

### 製品

<table>
<thead>
<tr>
<th align="left">Ryzen 4000 U/H</th>
<th align="center">4800U</th>
<th align="center">4800H</th>
<th align="center">4700U</th>
<th align="center">4600U</th>
<th align="center">4600H</th>
<th align="center">4500U</th>
<th align="center">4300U</th>
</tr>
</thead>

<tbody>
<tr>
<td align="left">CPU Core / Thread</td>
<td align="center" colspan="2">8/16</td>
<td align="center">8/8</td>
<td align="center" colspan="2">6/12</td>
<td align="center">6/6</td>
<td align="center">4/4</td>
</tr>

<tr>
<td align="left">CPU Base Clock (GHz)</td>
<td align="center">1.8</td>
<td align="center">2.9</td>
<td align="center">2.0</td>
<td align="center">?</td>
<td align="center">3.0</td>
<td align="center">2.3</td>
<td align="center">2.7</td>
</tr>

<tr>
<td align="left">CPU Boost Clock (GHz)</td>
<td align="center" colspan="2">4.2</td>
<td align="center">4.1</td>
<td align="center" colspan="2">4.0</td>
<td align="center">4.0</td>
<td align="center">3.7</td>
</tr>

<tr>
<td align="left">Total CPU L2$</td>
<td align="center" colspan="3">4MB</td>
<td align="center" colspan="3">3MB</td>
<td align="center">2MB</td>
</tr>

<tr>
<td align="left">Total CPU L3$</td>
<td align="center" colspan="6">8MB</td>
<td align="center">4MB</td>
</tr>

<tr>
<td align="left">GPU Core</td>
<td align="center">8</td>
<td align="center" colspan="2">7</td>
<td align="center" colspan="3">6</td>
<td align="center">5</td>
</tr>

<tr>
<td align="left">GPU Clock (MHz)</td>
<td align="center">1750</td>
<td align="center" colspan="2">1600</td>
<td align="center" colspan="3">1500</td>
<td align="center">1400</td>
</tr>

<tr>
<td align="left">Default TDP</td>
<td align="center">15W</td>
<td align="center">45W</td>
<td align="center">15W</td>
<td align="center">15W</td>
<td align="center">45W</td>
<td align="center">15W</td>
<td align="center">15W</td>
</tr>

<tr>
<td align="left">cTDP</td>
<td align="center">10-25W</td>
<td align="center">35-54W</td>
<td align="center">10-25W</td>
<td align="center">10-25W</td>
<td align="center">35-54W</td>
<td align="center">10-25W</td>
<td align="center">10-25W</td>
</tr>
</tbody>
</table>

4800UのBase Clockが1.8 GHzと他よりも低くなっているが、CPUが8-Core/16-Thread、GPUも4000 U/H-seriesでは最高性能でありながらTDP15Wなため仕方ない。  
むしろPicassoの4C/8T、Base 2.3 GHzからコア数を倍にしても0.5 GHzの低下に抑えているあたりTSMC 7nm FinFetの恩恵を窺える。  

4300Uは最も低いモデルとなり、Base ClockこそH-seriesに迫るほどだが、代わりに4C/4Tであり、L3キャッシュが4MBとのことからCCXを1つ丸々無効化しているはずだ。  
近いBase ClockはPicasso 3000 seriesでは2C/4Tでないとないため、4300Uも7nmの恩恵を十分に受けているとも考えられる。  
SMTを有効にした4C/8Tモデルがないのは、Picasso 3000 seriesの上位製品と競合しないようにするためかもしれない。（4000 seriesが出てもしばらくはPicassoを採用した新規製品が投入されるはず）  

### 構造
![Renoir Diagram](/image/2020/01/07/renoir-diagram.webp)  
いつものダイアグラム。  

#### プロセス
他のAMD 7nm製品と同様にTSMC 7nm FinFetで製造されている。  
AMDは、Cinebench R20の結果においてRyzen 7 4800UはRyzen 7 3700Uの2倍の電力効率を実現したとし、  
内70%は7nmプロセスによるものと語る。  

#### CPU
アーキテクチャはZen2ではあるが、特徴の１つであった大容量L3キャッシュはなくなり、コアあたり1MBとRaven/Picassoと同じ容量となった。  
ただこれは、そもそもZen2 CCDでL3キャッシュをZen/+から倍に増やしたのは、チップレットデザインによるダイ間アクセスのレイテンシを隠蔽するためとAMDは語っており、1チップに収めたRenoirで大容量L3キャッシュは不要どころかコスト、消費電力的に邪魔になると判断したのだろう。  

[AMD Ryzen Threadripper 3970X review - The Threadripper Processor Series - Guru3D](https://www.guru3d.com/articles_pages/amd_ryzen_threadripper_3970x_review,2.html)  

>  Why the double L3 cache? Well, AMD needed to address the latencies for accessing working memory to cope with the chiplet design, whereby the memory controller is physically located in a different chip, ergo a doubled L3 cache. 

#### GPU
最大8CUになり、それ以外のROPやL2キャッシュ等は不明。  
アーキテクチャは引き続きVega（GFX9）となるが、細かいことを言うとGPU IDはRaven/Picassoのgfx902ではなく、Raven2と同じgfx909であり、gfx909ではgfx902にあったバグが修正されている。  
クロックはPicassoの最大1400MHzから最大1800MHzまで上昇した。  
性能に関しては後述。  

#### I/O
メモリコントローラーがLPDDR4x-4266に対応、PCIeはGen3に留まること以外は不明。  

完全に推測だが、RavenからI/Oに関しては増やしてないのではないかと思う。  
微細化によってCPU/GPU部が小さくなっても、I/Oのアナログ部はほとんど前プロセスと同じとなる。  
そのためI/Oを増やすとダイサイズをも増加させてしまい、かといって減らすと製品として見劣りしてしまう。  
そういうことで増やしてないと考えた。  

PCIeがGen3となったのは、やはりGen4の発熱、消費電力はモバイル向けには受け入れられづらく、またモバイル向けに使えるGen4対応機器がまだNVMe SSDくらいしかないからだろう。  
モバイル向けに広帯域が求められるチップ/デバイスを接続することはあまりないし、あると分かっているならUSB (10Gbps)やIcelakeのTB3コントローラーのようにSoCの機能として内蔵してしまった方が消費電力が削減できる。  

#### マルチメディア
それまでのVCN 1.0からNavi1xと同じVCN 2.0に置き換わり、  
HEVC、VP9 4K90/8K24デコード、HEVC 4K60エンコードに対応する。  
VP9デコードはVCN 1.0で対応していたが、最大解像度は4Kまでだった。  
ディスプレイ部はDCN 2.1となり、Linux Kernelのドライバーを見ると最大出力数4、3x DSCとなっている。  

#### その他
Infinity Fabricのクロックがメモリクロックに同期しないようにしたらしく、これによってアイドル時の消費電力が削減された。  
関係あるかは分からないが、cTDPの下限が10Wと、Raven/Picassoの12Wより下げることが可能となっている。  

### 性能
TDP15WでGPUが最高性能となるPicasso(Winston) Ryzen 7 3780Uと、Renoir Ryzen 7 4800Uを比較した表が以下。  

| Zen APU | 3780U | 4800U |
| :--- | :---: | :---: |
| CPU Core/Thread | 4/8 | 8/16 |
| CPU Base Clock (GHz) | 2.3 | 1.8 |
| CPU Boost Clock (GHz) | 4.0 | 4.2 |
||
| GPU CUs | 11 | 8 |
| GPU SPs | 704 | 512 |
| GPU TMUs | 44 | 32 |
| GPU ROPs | 8 | 8 ? |
| GPU Clock (MHz) | 1400 | 1750 |
||
| Memory Type | DDR4 | LPDDR4x |
| Memory Speed (MT/s) | 3200 | 4266 |
| Memory Bandwidth (GB/s) | 51.2 | 68.3 |
||
| FP16 (FLOPS) | 3.94 | 3.58 |
| FP32 (FLOPS) | 1.97 | 1.79 |

Surface専用の3780Uではなく、3700Uとの比較だとメモリ帯域の差はさらに広まり（38.4 GB/s vs 68.3 GB/s）、FP32ピーク性能は4800Uと一致する。（10CU 1400MHz）  

RenoirでGPUクロックが300-400MHz向上したが、CUを減らした分をギリギリ補えなるくらいで、フルスペックでの比較だとコンピュート性能は下がってしまっている。  
が、統合GPUにおける最大のボトルネックはメモリ帯域であり、そこを伸ばすのが性能向上における効率が最も良い。  
仮にカラー圧縮で帯域削減、L1キャッシュ増設をしたRDNAアーキテクチャをAPUに取り入れてボトルネックの軽減に役立てても、それは効率の悪い部類に入り、L1キャッシュといったSRAMはダイサイズを増加させ、コスト、消費電力をも増やす要因となるため、やはり効率が悪い。  

（追記 2020-01-08-18:23）
とか書いたけどAMDのSenior Techical Marketing ManagerであるRobert Hallock氏によるとスケジュールの都合によるものらしい。  

そしてRenoirは第二世代のRyzen Mobile、PicassoのCUより59%高速とのこと。  
クロック向上とLPDDR4x-4266によるものだろうか。  
<https://twitter.com/Thracks/status/1215137876922396672>  
（追記終了）

#### vs Icelake (Gen11 GT2)
AMDは、Ryzen 7 4800U と Core i7 1065G7 の性能比較において、4800Uの方がシングルスレッド性能では4%、マルチスレッド性能では90%、グラフィック性能では18%上だと発表している。[^1]  
Core i7 1065G7 を搭載したDell XPS 7390を用いており、メモリはLPDDR4x-3733だろう。  
2つのTDP設定は不明。どちらもデフォルトで15W、cTDPで最大25Wであるから、そのどっちかではあるはずだ。  

 > The new AMD Ryzen 7 4800U offers:  
Up to 4% greater single-thread performance and up to 90% faster multithreaded performance than the competition<sup>8</sup>  
Up to 18% faster graphics performance than the competition<sup>9</sup>

[^1]: [AMD Announces World’s Highest Performance Desktop and Ultrathin Laptop Processors at CES 2020 - AMD](http://ir.amd.com/news-releases/news-release-details/amd-announces-worlds-highest-performance-desktop-and-ultrathin)

スペックでは、RenoirのクロックがGen11 GT2の x1.59でありながらグラフィック性能が x1.18に留まっている。  
メモリ帯域は x1.14であり、概ねこれに沿っていると見られる。  
やはりメモリ帯域が統合GPUの
実行性能で支配的にあるのだろう。  

#### vs Tigerlake (Gen12LP) ?
TigerlakeのGPU部、Gen12LPではEU/Shading UnitsがGen11 GT2から x1.5され、メモリもLPDDR5-6400を使用すればLPDDR4-3733から x1.71の帯域が手に入る。  
Intel自身、CES2020のプレスカンファレンス後のニュースリリースにて、  
CPU、AIアクセラレーター、X<sup>e</sup>グラフィクスアーキテクチャーの最適化によってTigerlakeは二桁台の性能向上を実現すると発表している。[^2]  

 > With optimizations spanning the CPU,  
AI accelerators and discrete-level integrated graphics based on the new Intel Xe graphics architecture,  
Tiger Lake will deliver double-digit performance gains<sup>1</sup>,  
massive AI performance improvements,  
a huge leap in graphics performance and 4x the throughput of USB 3 with the new integrated Thunderbolt 4.  
Built on Intel’s 10nm+ process, the first Tiger Lake systems are expected to ship this year.

[^2]: [2020 CES: Intel Brings Innovation to Life with Intelligent Tech Spanning the Cloud, Network, Edge and PC - Intel Newsroom](https://newsroom.intel.com/news-releases/intel-ces-2020/)  

3つまとめてな上、二桁台と曖昧で怪しさ満点というか、まだ詳細隠しておきたい意図が見られるが、  
マルチスレッド性能は無理としても、シングルスレッド性能とグラフィック性能ではRenoirを追い越せるのではないかと思う。  
シングルスレッド性能はIcelakeとRenoirであまり差がないことから、クロック特性さえ改善されCPUクロックが0.3GHzほど向上すればいい、グラフィック性能はLPDDR5-6400だとLPDDR4X-4266の x1.5のメモリ帯域があることからそう考えた。  

<span style="color:dodgerblue">Icelake</span> が <span style="color:coral">Picasso</span> を抜かし、 <span style="color:coral">Renoir</span> が <span style="color:dodgerblue">Icelake</span> を抜かし、<span style="color:dodgerblue">Tigerlake</span> が <span style="color:coral">Renoir</span>を抜かすCPU/iGPUのデッドヒート。  

ただもっと怪しい部分があり、TigerlakeはIntelの10nm+ プロセスで製造されるとのことだが、その10nm+というのはIcelakeに使われているのと同じということだ。  

[Ice Lake Processor Family - Intel](https://www.intel.com/content/www/us/en/design/products-and-solutions/processors-and-chipsets/ice-lake/overview.html)  

 > The Ice Lake processor family is the next generation Intel® Core™ processor family. These processors utilize Intel’s industry-leading 10 nm+ process technology.

Tigerlakeの中身（特にCPU）がどうなってるのか、靄がかかり始める。  
2020年中に製品が出るとのことだがどうなるか。  
まあ気長に待つ他ない。  

以前X<sup>e</sup>、DG1の推測に使用した表に一部修正を加えたもの。  

| Integral GPU | Intel Gen11 GT2 | Intel Gen12LP | AMD Picasso (3700U) | AMD Renoir (4800U) |
| :-- | :--: | :--: | :--: | :--: |
| GPU Clock | 1.1 GHz | 1.1 GHz~? | 1.4 GHz | 1.75 GHz |
| Shading Units | 512 | 768 | 640 | 512 |
| TMUs | 32 | 24 ?? | 40 | 32 |
| ROPs | 16 | 16 ? | 8 | 8 ? |
| GPU $ | 3MB | 3MB? | 1MB | 1MB? | 
||
| Memory Type | LPDDR4/x | LPDDR4x /LPDDR5 | DDR4 | LPDDR4/x |
| Memory Speed | 3733 MT/s | ~6400 MT/s? | 2400 MT/s | 4266 MT/s |
| Memory Bandwidth | 59.7 GB/s | 102.4 GB/s | 38.4 GB/s | 68.3 GB/s | 
||
| Peak Texture Fill-Rate (GT/s) | 35.2 | 26.4~? | 56.0 | 56.0 |
| Peak Pixel Fill-Rate (GP/s) | 17.6 | 17.6~? | 11.2 | 14.0 |
| FP16 (TFLOPS) | 2.2 | 3.4~? | 3.58 | 3.58 |
| FP32 (TFLOPS) | 1.1 | 1.7~? | 1.79 | 1.79 |

<br>

<table>
<thead>
<tr>
<th align="left"><span style="color:crimson">RV Family</th>
<th align="center"><span style="color:coral">Raven</th>
<th align="center"><span style="color:coral">Raven2</th>
<th align="center"><span style="color:coral">Picasso</th>
<th align="center"><span style="color:coral">Renoir</th>
<th align="center"><span style="color:#f4a460">Dali</th>
</tr>
</thead>

<tbody>
<tr>
<td align="left">CMOS CPU</td>
<td align="center" colspan="2">14nm</td>
<td align="center">12nm</td>
<td align="center">7nm</td>
<td align="center">12nm?</td>
</tr>

<tr>
<td align="left">CPU</td>
<td align="center" colspan="2">Zen(+)</td>
<td align="center">Zen+</td>
<td align="center">Zen2</td>
<td align="center">Zen+?</td>
</tr>

<tr>
<td align="left">Max CPU Core/Thread</td>
<td align="center">4/8</td>
<td align="center">2/4</td>
<td align="center">4/8</td>
<td align="center">8/16</td>
<td align="center">2/4 ?</td>
</tr>

<tr>
<td align="left">L3$ CPU</td>
<td align="center" colspan="3">4MB</td>
<td align="center">8MB</td>
<td align="center">4MB?</td>
</tr>

<tr>
<td align="left"></td>
<td align="center" colspan="5"></td>
</tr>

<tr>
<td align="left">CMOS GPU</td>
<td align="center" colspan="2">14nm</td>
<td align="center">12nm</td>
<td align="center">7nm</td>
<td align="center">12nm?</td>
</tr>

<tr>
<td align="left">GPU</td>
<td align="center" colspan="5">Vega (GFX9)</td>
</tr>

<tr>
<td align="left">GPU Clock</td>
<td align="center">1300 MHz</td>
<td align="center">1200 MHz</td>
<td align="center">1400 MHz</td>
<td align="center">1800 MHz</td>
<td align="center"></td>
</tr>

<tr>
<td align="left">Shader Engine</td>
<td align="center" colspan="3">1</td>
<td align="center">1 ?</td>
<td align="center">1 ?</td>
</tr>

<tr>
<td align="left">Max CUs</td>
<td align="center">11</td>
<td align="center">3</td>
<td align="center">11</td>
<td align="center">8 ?</td>
<td align="center">3 ?</td>
</tr>

<tr>
<td align="left">Max TMUs</td>
<td align="center">44</td>
<td align="center">12</td>
<td align="center">44</td>
<td align="center">32 ?</td>
<td align="center">12 ?</td>
</tr>

<tr>
<td align="left">Max ROPs</td>
<td align="center">8</td>
<td align="center">4</td>
<td align="center">8</td>
<td align="center">8 ?</td>
<td align="center">4 ?</td>
</tr>

<tr>
<td align="left">L2$ GPU</td>
<td align="center">1 MB</td>
<td align="center">0.5 MB</td>
<td align="center">1 MB</td>
<td align="center">1 MB?</td>
<td align="center">0.5 MB?</td>
</tr>

<tr>
<td align="left"></td>
<td align="center"><span style="color:coral">Raven</td>
<td align="center"><span style="color:coral">Raven2</td>
<td align="center"><span style="color:coral">Picasso</td>
<td align="center"><span style="color:coral">Renoir</td>
<td align="center"><span style="color:#f4a460">Dali</td>
</tr>

<tr>
<td align="left">Memory Type</td>
<td align="center" colspan="3">DDR4</td>
<td align="center">LP/DDR4/X</td>
<td align="center">DDR4 ?</td>
</tr>

<tr>
<td align="left">Support Memory Speed</td>
<td align="center">2933 MHz</td>
<td align="center">2400 MHz</td>
<td align="center">2933 MHz</td>
<td align="center">4266 MHz</td>
<td align="center">2933 MHz?</td>
</tr>

<tr>
<td align="left">VCN ver</td>
<td align="center" colspan="3">1.0</td>
<td align="center">2.0</td>
<td align="center">1.0?</td>
</tr>

<tr>
<td align="left">DCN ver</td>
<td align="center" colspan="3">1.0</td>
<td align="center">2.1</td>
<td align="center">1.0</td>
</tr>

<tr>
<td align="left">DeviceID</td>
<td align="center">15DD</td>
<td align="center">15D8 /15DD</td>
<td align="center">15D8</td>
<td align="center">1636</td>
<td align="center">15D8 /15D9?</td>
</tr>

<tr>
<td align="left">GPU ID</td>
<td align="center">gfx902</td>
<td align="center">gfx909</td>
<td align="center">gfx902</td>
<td align="center">gfx909</td>
<td align="center">gfx909</td>
</tr>

<tr>
<td align="left">Die Size</td>
<td align="center">209.78 mm<sup>2</sup></td>
<td align="center">154.68 mm<sup>2</sup>??</td>
<td align="center">209.78 mm<sup>2</sup></td>
<td align="center">~160mm<sup>2</sup>?</td>
<td align="center"></td>
</tr>
</tbody>
</table>

<hr>
<code>Renoir関連</code>

 * [Renoirの構造推測 ](/posts/2019/11/09/renoir-guess/)  
 * [Renoirの構造推測 Part2 ](/posts/2019/12/02/renoir-guess-p2/)  
 * [Renoirの情報アップデート&推測 Part3 ](/posts/2019/12/21/renoir-guess-p3/)  
 * [CES2020にてAMDよりRyzen 4000 U/H-Series発表 & スペック詳細](/posts/2020/01/07/ces2020-p1)
