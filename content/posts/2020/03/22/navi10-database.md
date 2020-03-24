---
title: "Navi10 Database"
date: 2020-03-23T09:46:48+09:00
draft: true
tags: [ "Radeon", "RDNA", "GFX10", "gfx1010", "Navi10", "Database" ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
---

あるGPUについての情報を集めていく内に、細かい情報が疎になってしまっていたため、集約した記事を作成することとした。  
今後情報に更新があった場合は、単独記事を作成すると同時に、この記事も更新していく。  

| AMDGPU | Navi10 |
| :--- | :---: |
| &ensp;GFX Core | GFX10 |
| &ensp;GPU ID | gfx1010 |
| <span style="color:#FF8C00">Spec</span>|
| &ensp;Shader Engine | 2 |
| &ensp;&mdash; Shader Array | 4 |
| &ensp;&mdash;&mdash; WGP | 20 |
| &ensp;&mdash;&mdash;&mdash; CU | 40 |
| &ensp;&mdash;&mdash;&mdash;&mdash; SP | 2560 |
| &ensp;&mdash;&mdash;&mdash;&mdash; TMU | 160 |
| &ensp;ROP | 64<br>(16x Render Backend)|
| &ensp;&mdash;&mdash;&mdash; I$<br>(Instruction Cache) | 640 KB<br>(20x 32 KB)
| &ensp;&mdash;&mdash;&mdash; K$<br>(Scalar Cache) | 320 KB<br>(20x 16 KB)
| &ensp;&mdash;&mdash;&mdash;&mdash; L0 Cache | 520 KB<br>(40x 16 KB)
| &ensp;&mdash;&mdash; L1 Cache | 512 KB<br>(4x 128 KB)
| &ensp;&mdash; L2 Cache | 4 MB<br>(16x 256 KB) |
| &ensp;L0$-L1$ Bus Width | Load: 128 Byte/Cycle<br>Store: 64 B/C |
| &ensp;(I$ K$)-L1$ | Load/Store:<br>32 B/C |
| &ensp;L1$-L2$ Bus Width | Load/Store:<br>64 B/C |
| &ensp;Memory Interface | 256-bit |
| &ensp;Memory Bandwidth | 448 GB/s<br>(14Gbps) |
| <span style="color:#FF8C00">Multi Media</span> |
| &ensp;Display Core | DCN 2.0 |
| &ensp;Display Controller | 6 |
| &ensp;VCN Core | VCN 2.0 |
| <span style="color:#FF8C00">I/O</span> |
| &ensp;PCIe | Gen4 16-Lane<br>(31.5 GB/s) |
| <span style="color:#FF8C00">Other</span> |
| &ensp;Process | TSMC 7nm FinFet |
| &ensp;Transistor Count | 10.3 B |
| &ensp;Die Size | 251 mm<sup>2</sup> |
| &ensp;First Linux Kernel Patch | 2019/06/17[^1] |

[^1]: [[PATCH 000/459] amdgpu support for Navi10](https://lists.freedesktop.org/archives/amd-gfx/2019-June/035170.html)

{{< figure src="/image/2020/01/22/navi10-dieshot-guess.webp" title="Navi10 ダイアグラム推測" caption="Update: 2020-03-23<br>画像元: <cite>[AMD@7nm@RDNA\_1th\_gen@Navi10@Radeon\_RX\_5700\_XT@215-0917210@\_\_\_DSCx2@NIR](https://www.flickr.com/photos/130561288@N04/49411586768/)</cite>" >}}

## 特筆点
### USB Type-C Power Delivery
内部にUSB Type-Cコントローラを備え、Navi10ベース製品の中では2020/03/22時点で Radeon Pro W5700 のみにコネクタが実装されている。  
ダイショットを見る限り、2基の Display PHY が他4基とは離されているが、これはUSB Type-Cから映像出力を行なうためと思われる。  

### ドット積混合演算には一部対応せず
*Navi12/gfx1011* 、*Navi14/gfx1012* では対応している一部ドット積の混合演算命令に *Navi10/gfx1010* は対応していない。[^2]  

[^2]: <https://github.com/llvm-mirror/llvm/blob/master/lib/Target/AMDGPU/AMDGPU.td#L846>  

ドット積の混合演算は、深層学習における推論の高速化に用いられることをAMDは想定している。  

 > Some variants of the dual compute unit expose additional mixed-precision dot-product modesin the ALUs, primarily for accelerating machine learning inference.

 > 引用元: <cite><https://www.amd.com/system/files/documents/rdna-whitepaper.pdf#page=14></cite>

対応しなかった理由としては、設計時期か、それに合わせて *Navi10* をグラフィック/ゲーミング市場に出すため、ターゲットではなかったことから敢えて省いた、等が考えられる。  


## Navi10ベース製品 

### Gaming Desktop Product

 * [AMD Radeon™ RX 5600 Graphics | AMD](https://www.amd.com/en/products/graphics/amd-radeon-rx-5600#product-specs)
 * [AMD Radeon™ RX 5600 XT Graphics | AMD](https://www.amd.com/en/products/graphics/amd-radeon-rx-5600-xt#product-specs)
 * [AMD Radeon™ RX 5700 Graphics | AMD](https://www.amd.com/en/products/graphics/amd-radeon-rx-5700#product-specs)
 * [Radeon™ RX 5700 XT Graphics | AMD](https://www.amd.com/en/products/graphics/amd-radeon-rx-5700-xt#product-specs)
 * [AMD Radeon™ RX 5700 XT 50th Anniversary Graphics | AMD](https://www.amd.com/en/products/graphics/amd-radeon-rx-5700-xt-50th-anniversary#product-specs)

<details><summary>Gaming Desktop Spec</summary>
<table>
<thead>
<tr>
<th align="left">Gaming Desktop</th>
<th align="center">RX 5600</th>
<th align="center">RX 5600 XT</th>
<th align="center">RX 5700</th>
<th align="center">RX 5700 XT</th>
<th align="center">RX 5700 XT 50th</th>
</tr>
</thead>

<tbody>
<tr>
<td align="left">WGP</td>
<td align="center">16</td>
<td align="center" colspan="3">18</td>
<td align="center">20</td>
</tr>

<tr>
<td align="left">&mdash; CU</td>
<td align="center">32</td>
<td align="center" colspan="3">36</td>
<td align="center">40</td>
</tr>

<tr>
<td align="left">&mdash;&mdash; SP</td>
<td align="center">2048</td>
<td align="center" colspan="3">2304</td>
<td align="center">2560</td>
</tr>

<tr>
<td align="left"></td>
<td align="center"></td>
<td align="center"></td>
<td align="center"></td>
<td align="center"></td>
<td align="center"></td>
</tr>

<tr>
<td align="left">Game Clock</td>
<td align="center" colspan="2">1375 MHz</td>
<td align="center">1625 MHz</td>
<td align="center">1755 MHz</td>
<td align="center">1830 MHz</td>
</tr>

<tr>
<td align="left">Boost Clock</td>
<td align="center" colspan="2">1560 MHz</td>
<td align="center">1720 MHz</td>
<td align="center">1905 MHz</td>
<td align="center">1980 MHz</td>
</tr>

<tr>
<td align="left">Memory Size</td>
<td align="center" colspan="2">6 GB</td>
<td align="center" colspan="4">8 GB</td>
</tr>

<tr>
<td align="left">Memory Interface</td>
<td align="center" colspan="2">192-bit</td>
<td align="center" colspan="4">256-bit</td>
</tr>

<tr>
<td align="left">Memory Bandwidth (GB/s)</td>
<td align="center">288</td>
<td align="center">288(12Gbps)<br>336(14Gbps)</td>
<td align="center" colspan="3">448</td>
</tr>

<tr>
<td align="left"></td>
<td align="center"></td>
<td align="center"></td>
<td align="center"></td>
<td align="center"></td>
<td align="center"></td>
</tr>

<tr>
<td align="left">Peak FP32 (TFLOPS)</td>
<td align="center">6.39</td>
<td align="center">7.19</td>
<td align="center">7.95</td>
<td align="center">9.75</td>
<td align="center">10.14</td>
</tr>

<tr>
<td align="left">Typical Board Power</td>
<td align="center" colspan="2">150 W</td>
<td align="center">180 W</td>
<td align="center">225 W</td>
<td align="center">235 W</td>
</tr>

<tr>
<td align="left">Launch Date</td>
<td align="center" colspan="2">2020/01/06</td>
<td align="center" colspan="3">2019/07/07</td>
</tr>

<tr>
<td align="left">SKU</td>
<td align="center">Navi10 XE</td>
<td align="center">Navi10 XLE</td>
<td align="center">Navi10 XL</td>
<td align="center">Navi10 XT</td>
<td align="center">Navi10 XTX</td>
</tr>
</tbody>
</table>
</details>

### Gaming Mobile Product

 * [AMD Radeon™ RX 5600M | AMD](https://www.amd.com/en/product/9031)
 * [AMD Radeon™ RX 5700M | AMD](https://www.amd.com/en/product/9016)

<details><summary>Gaming Mobile Spec</summary>
<table>
<thead>
<tr>
<th align="left">Gaming Mobile</th>
<th align="center">RX 5600M</th>
<th align="center">RX 5700M</th>
</tr>
</thead>

<tbody>
<tr>
<td align="left">WGP</td>
<td align="center" colspan="2">18</td>
</tr>

<tr>
<td align="left">&mdash;&mdash; CU</td>
<td align="center" colspan="2">36</td>
</tr>

<tr>
<td align="left">&mdash;&mdash; SP</td>
<td align="center" colspan="2">2304</td>
</tr>

<tr>
<td align="left"></td>
<td align="center"></td>
<td align="center"></td>
</tr>

<tr>
<td align="left">Game Clock</td>
<td align="center">1190 MHz</td>
<td align="center">1620 MHz</td>
</tr>

<tr>
<td align="left">Boost Clock</td>
<td align="center">1265 MHz</td>
<td align="center">1720</td>
</tr>

<tr>
<td align="left">Memory Size</td>
<td align="center">6 GB</td>
<td align="center">8 GB</td>
</tr>

<tr>
<td align="left">Memory Interface</td>
<td align="center">192-bit</td>
<td align="center">256-bit</td>
</tr>

<tr>
<td align="left">Memory Bandwidth (GB/s)</td>
<td align="center">288</td>
<td align="center">384</td>
</tr>

<tr>
<td align="left"></td>
<td align="center"></td>
<td align="center"></td>
</tr>

<tr>
<td align="left">Peak FP32 (TFLOPS)</td>
<td align="center">5.83</td>
<td align="center">7.93</td>
</tr>

<tr>
<td align="left">TBP</td>
<td align="center" colspan="2">?</td>
</tr>

<tr>
<td align="left">Launch Date</td>
<td align="center" colspan="2">2020/01/06</td>
</tr>

<tr>
<td align="left">SKU</td>
<td align="center">Navi10 XME?</td>
<td align="center">?</td>
</tr>
</tbody>
</table>
</details>

### Pro Desktop Product

 * [Radeon™ Pro W5700 | Workstation Graphics Card | AMD](https://www.amd.com/en/products/professional-graphics/radeon-pro-w5700#product-specs)
 * Radeon Pro W5700X

<details><summary>Pro Desktop</summary>
<table>
<thead>
<tr>
<th align="left">Pro Desktop</th>
<th align="center">Radeon Pro W5700</th>
<th align="center">Radeon Pro W5700X</th>
</tr>
</thead>

<tbody>
<tr>
<td align="left">WGP</td>
<td align="center">18</td>
<td align="center">20</td>
</tr>

<tr>
<td align="left">&mdash; CU</td>
<td align="center">36</td>
<td align="center">40</td>
</tr>

<tr>
<td align="left">&mdash;&mdash; SP</td>
<td align="center">2304</td>
<td align="center">2560</td>
</tr>

<tr>
<td align="left"></td>
<td align="center"></td>
<td align="center"></td>
</tr>

<tr>
<td align="left">Boost Clock</td>
<td align="center">1929 MHz</td>
<td align="center">1855 MHz</td>
</tr>

<tr>
<td align="left">Memory Size</td>
<td align="center">8 GB</td>
<td align="center">16 GB</td>
</tr>

<tr>
<td align="left">Memory Interface</td>
<td align="center" colspan="2">256-bit</td>
</tr>

<tr>
<td align="left">Memory Bandwidth (GB/s)</td>
<td align="center" colspan="2">448</td>
</tr>

<tr>
<td align="left"></td>
<td align="center"></td>
<td align="center"></td>
</tr>

<tr>
<td align="left">Peak FP32 (TFLOPS)</td>
<td align="center">8.89</td>
<td align="center">9.5</td>
</tr>

<tr>
<td align="left">TBP</td>
<td align="center">190 W<br>205 W(+USB-C)</td>
<td align="center">?</td>
</tr>

<tr>
<td align="left">Launch Date</td>
<td align="center">2019/11/19</td>
<td align="center"></td>
</tr>

<tr>
<td align="left">SKU</td>
<td align="center">Navi10 Pro XL?</td>
<td align="center">?</td>
</tr>
</tbody>
</table>
</details>

<hr>
## 参考資料

 * [pal/ndDevice.cpp at dev · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/dev/src/core/os/nullDevice/ndDevice.cpp)
	 * <https://github.com/GPUOpen-Drivers/pal/blob/dev/src/core/os/nullDevice/ndDevice.cpp#L922>
 * [RadeonFeature](https://www.x.org/wiki/RadeonFeature/)
 * [llvm/AMDGPU.td at master · llvm-mirror/llvm](https://github.com/llvm-mirror/llvm/blob/master/lib/Target/AMDGPU/AMDGPU.td)
 	* <https://github.com/llvm-mirror/llvm/blob/master/lib/Target/AMDGPU/AMDGPU.td#L846>
 * [Discovering the structure of RDNA - GPUOpen](https://gpuopen.com/discovering-rdna/)
 	* <https://gpuopen.com/wp-content/uploads/2019/08/RDNA_Architecture_public.pdf>
 	* <https://www.amd.com/system/files/documents/rdna-whitepaper.pdf>
 * [AMD RDNA 1.0 Instruction Set Architecture - GPUOpen](https://gpuopen.com/compute-product/amd-rdna-1-0-instruction-set-architecture/)
 	* <https://gpuopen.com/wp-content/uploads/2019/08/RDNA_Shader_ISA_5August2019.pdf>
	* <https://developer.amd.com/wp-content/resources/RDNA_Shader_ISA.pdf>
