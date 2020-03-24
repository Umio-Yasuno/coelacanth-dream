---
title: "Navi12情報近況 (2020-03-09) & 推測"
date: 2020-03-09T04:16:47+09:00
draft: false
tags: [ "gfx1011", "Navi12", "RDNA", "GFX10" ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "GPU" ]
noindex: false
---

相変わらず実物が出てこない *Navi12* 。そうこうしている内に次世代GPU [RDNA 2](/tags/rdna-2) の実シリコンが動作しているという話が出てきてしまった。（DXR 1.1の実行結果の画像1枚ではあるが）  
しかしソフトウェア面ではちゃっかり出てきており、最新のLinux版Radeon Proドライバには *Navi12* のファームウェア、そして amdgpu.ids に 7360:C1 が既に追加されている。  
[Radeon Pro Software for Enterprise 20.Q1.1 for Linux Release Notes | AMD](https://www.amd.com/en/support/kb/release-notes/rn-pro-lin-20-q1-1)  
期待を捨てずに情報を追っていきたい。  

## より広範な混合演算に対応
*Navi12* 及び *Navi14* 、*Arcturus* は、*Vega20* よりも広範なドット積の混合演算に対応する。  
ドット積の混合演算は機械学習の推論高速化に用いられる。[^2]  

[^2]: [Introduction - rdna-whitepaper.pdf - Page 13](https://www.amd.com/system/files/documents/rdna-whitepaper.pdf#page=13)

LLVMの記述に合わせて言えば、*Vega20* では `dot1-insts`、`dot2-insts` までの対応だったのが、  
*Navi12* 、*Navi14* では `dot5-insts`、`dot6-insts` が、  
*Arcturus* では `dot3-insts`、`dot4-insts`、`dot5-insts`、`dot6-insts` の対応が追加された。  
ここでの `dot` の後の数字はバージョンを表すと思われる。  
[llvm-project/AMDGPU.cpp at master · llvm/llvm-project](https://github.com/llvm/llvm-project/blob/master/clang/lib/Basic/Targets/AMDGPU.cpp#L141)  

各々が含む命令は以下

 > * dot1-insts
 	* v_dot4_i32_i8
	* v_dot8_i32_i4
* dot2-insts
	* v_dot2_f32_f16
	* v_dot2_i32_i16
	* v_dot2_u32_u16
	* v_dot4_u32_u8
	* v_dot8_u32_u4
* dot3-insts
	* v_dot8c_i32_i4
* dot4-insts
	* v_dot2c_i32_i16
* dot5-insts
	* v_dot2c_f32_f16
* dot6-insts
	* v_dot4c_i32_i8
 >
 > 引用元: <cite>[llvm/AMDGPU.td at master · llvm-mirror/llvm](https://github.com/llvm-mirror/llvm/blob/master/lib/Target/AMDGPU/AMDGPU.td#L390)</cite>  

`dot[3-6]-insts` はエンコード先が追加されたものと読めるが、自信はあんまない  
[llvm-project/xdl-insts-gfx908.txt at master · llvm/llvm-project](https://github.com/llvm/llvm-project/blob/master/llvm/test/MC/Disassembler/AMDGPU/xdl-insts-gfx908.txt)  
[llvm-project/xdl-insts-gfx1011-gfx1012.txt at master · llvm/llvm-project](https://github.com/llvm/llvm-project/blob/master/llvm/test/MC/Disassembler/AMDGPU/xdl-insts-gfx1011-gfx1012.txt)  

*Navi10* は上記命令には対応しておらず、そのためか *Navi12 (gfx1011)* は CLRX では *NaviDL* とされたりもしている。[^1]  
*Vega20* 、*Navi14* でも対応していたのに *Navi10* で見送ったのは開発時期によるものだろうか。  

[^1]: [CLRadeonExtender: Add new GPU architecture GFX1011 (Navi with DLOps). · CLRX/CLRX-mirror@a4c9fdf](https://github.com/CLRX/CLRX-mirror/commit/a4c9fdfd191eda8fb206debe778dc9130caa3545)

## Navi12はクラウドゲーミングに採用されるか
[前の記事](/posts/2020/03/06/amd-financial-analyst-day-2020/)でも少し触れたネタ。  

{{< figure src="/image/2020/03/06/amd-financial-analyst-day-2020_12.webp" title="AMD RDNA Architecture" caption="画像元: <cite>[FINANCIAL ANALYST DAY 2020 - David Wang: Driving GPU Leadership](https://ir.amd.com/static-files/321c4810-ffe2-4d6c-863f-690464c033a9)<cite>" >}}

先日のFinancial Analyst Day 2020でAMDは、RDNAアーキテクチャの特徴を紹介していたが、その中でモバイルからクラウドゲーミングまで広い性能帯をカバーできるとしていた。  
しかしRDNAアーキテクチャ、Navi10/12/14のどれかをベースにした製品でクラウドゲーミングサービスのサーバに採用された話は聞かない。  
単に性能としては可能だ、という意味かもしれないが、投資家向けの発表で実現していないものを出すかは微妙だと思う。少なくともほぼ確実な事柄を持ち出すのではないだろうか。  
そして**もし**、Navi1xベースの製品がクラウドゲーミングに採用されるならば、SR-IOV（ゲスト）用のDeviceIDを持ち、対応が進められている *Navi12* が最適なはずだ。  
Google StadiaにGPUが採用された際のプレスリリースでも、SR-IOVに基づいたハードウェアベースの機能を強調している。  
現状Google Stadiaは [Radeon Pro WX 8200](https://www.amd.com/en/products/professional-graphics/radeon-pro-wx-8200#product-specs) とほぼ同等の性能を持つカスタムGPUでクラウドサーバを構築している。  
GCNからRDNAで、電力効率が50%向上したことを考えれば *Navi12* ベースのカスタムGPUに置き換える可能性は十分にある……はず。  

以前だったら 2020-03-16 から開催予定の[GDC2020](https://gdconf.com/)で何かしらの発表を期待できたのだが、新型コロナウイルスの影響を受けて延期が決定されている。  
[3月開催予定だったゲーム開発者会議「GDC 2020」、新型コロナで今夏に延期 - ITmedia NEWS](https://www.itmedia.co.jp/news/articles/2002/29/news018.html)  

Navi12の姿はまた遠く？  

## 規模はNavi10と同一か
もう2ヶ月近く前の話だが、*Navi12 (gfx1011)* の[CompuBench](https://compubench.com/result.jsp)実行結果が現れた。  
それによると、*Navi12* は 18WGP (36CU)、最大コアクロック 1144MHz、VRAM 8GBとされている。  
1144MHzというのは以前Linux Kernelへのパッチで判明した、Peak(Game) Clock 1110MHzと近く、そこに不自然さはない。  
[[2/2] drm/amdgpu/smu: add peak profile support for navi12 - Patchwork](https://patchwork.freedesktop.org/patch/346277/)  
ネット上ではコアの仕様が *Navi10* と同じでは *Navi12* の存在意義は何か、ということで未だ確定していないVRAMがHBM2とする見向きがあるが、  
力及ばず、その根拠となるパッチやソースコードは見つけることができなかった。  

ただ上述の混合演算の対応を考えれば、*Navi12* は Navi10リフレッシュ とまではいかないが Navi10改 のような存在だと考えられなくもない  
競争力を取り戻すため *Navi10* の開発を急ぎ、コンシューマ向けGPUには求められない一部新機能は見送り、必要とされるサーバ向け等ではそれより遅れて出す *Navi12* でカバーする、としたの **かもしれない。**  

## ゲーミングMacに搭載か
AppleがゲーミングMacを発表するという *噂* があること、  
[ゲーミングMacが来年6月に最大55万円で発表されるかも!?](https://www.gizmodo.jp/2019/12/gaming-mac.html)  
Navi12の予約されているIDの中に、Apple向けに用いられる傾向の RevisionID: 4x があることから、  
Navi12はそれに搭載されるのではないか、という話がネット上では一定の支持を得てはいるが、噂に推測を重ねることは避けたいため、そういった話があることの紹介だけに留めたい。  
<span class="hide">散々妄想を垂れ流しておいてどの口が言うんだか……</span>

<hr>
<span class="reference">参考:</span>  

 * <cite>[drm/amdgpu: enable virtual display for navi12](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd?h=amd-staging-drm-next&id=7990202903135173893beb211b14d3a37e97a5e7)</cite>
