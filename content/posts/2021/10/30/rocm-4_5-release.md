---
title: "ROCm v4.5 がリリース ―― CPU+GPU の統合メモリをサポート、RDNA系のサポートは v5.0 に？"
date: 2021-10-30T12:19:55+09:00
draft: false
tags: [ "ROCm", ]
# keywords: [ "", ]
categories: [ "Software", "AMD", "GPU" ]
noindex: false
# summary: ""
---

AMD ROCm v4.5 がリリースされた。  
今リリースの大きな変更点としては、HIP (Heterogeneous-Compute Interface for Portability) において CPU+GPU の統合メモリをサポートしたことが挙げられる。  

 * [RadeonOpenCompute/ROCm at f088317e4483b8f75ec55ac3bb040f23e5abd2d4](https://github.com/RadeonOpenCompute/ROCm/tree/f088317e4483b8f75ec55ac3bb040f23e5abd2d4)

統合メモリは *Vega/GFX9* 世代とそれ以降の世代の GPU でサポートされ、*Polaris* や *Fiji* といった *GFX8* 世代の GPU ではサポートされない。  
これは、そもそも ROCm v4.0 から *GFX8* 世代は、動作はするが公式的に完全なサポートは保証しないことが明言されており、また *GFX8* 世代では 49-bitアドレッシングもサポートしていないことが関係していると思われる。  

統合メモリは 2種類の方法でサポートされ、XNACK-enabled と XNACK-disabled からなる。  
XNACK-disabled では、GPU が使うメモリすべてが GPU側のページテーブルにマッピングされている必要があり、ページフォールトが発生した際はドライバーが一時的に GPUキューに割り込み、それからページ移行の処理を行い、GPU は再度メモリアクセスを試みる。  
XNACK-enabled では GPU がページフォールトとページ移行の処理を行うことができるため、CPU+GPU の統合メモリにおいては XNACK-enabled の方が有利だと考えられる。  

しかしコンパイラは XNACK-enabled/disabled でそれぞれ別のコードを生成するため、現時点では XNACK-enabled は実験的なサポートとしている。  
XNACK を有効としてコンパイルされたコードを、XNACK が無効化されている GPU で実行すると、正しく実行されない、実行されても性能が低下する恐れがある。  
{{< link >}} [Linux Kernel に CPU + GPU の統合メモリ空間をサポートする最初のパッチが投稿される | Coelacanth's Dream](/posts/2021/01/07/add-svm-to-amdgpu-kfd/) {{< /link >}}

## 外されつつある Polaris と追加されつつある RDNA {#polaris-rdna}

上で既に公式的なサポートから *GFX8* 世代が外れていることには触れたが、一部 ROCmソフトウェア/ライブラリからは *Polaris (gfx803)* のサポートを *取り除く* 動きも見られる。  

 * [Drop gfx803 from default build architectures by cgmb · Pull Request #288 · ROCmSoftwarePlatform/rocSOLVER](https://github.com/ROCmSoftwarePlatform/rocSOLVER/pull/288)
 * [Revert "Drop gfx803 from default build architectures" by cgmb · Pull Request #330 · ROCmSoftwarePlatform/rocSOLVER](https://github.com/ROCmSoftwarePlatform/rocSOLVER/pull/330)
 * [Remove gfx803; Add gfx90a and gfx1030 by jithunnair-amd · Pull Request #835 · ROCmSoftwarePlatform/pytorch](https://github.com/ROCmSoftwarePlatform/pytorch/pull/835)
 * [[ROCm] Updating Dockerfile.rocm to drop support for gfx803 · tensorflow/tensorflow@a67c9b9](https://github.com/tensorflow/tensorflow/commit/a67c9b9a30815f7d602eb10563d1831e5863e062)

再度サポートが有効化されたり、プルリクエストが取り込まれなかったりでほとんどは *Polaris (gfx803)* のサポートを継続しているが、今後サポートする AMD GPU が増えた場合には今度こそ取り除かれると思われる。  

*Vega10 (gfx900)* についてもサポートから外される予兆を見せており、*Vega10* ベースのサーバー向け SKU **Instint MI25** は EOL (End of Life) に達し、ROCm v4.5 がサポートする最後の公式リリースになるとしている。[^mi25-eol]  
**Vega64** に対しては触れられていないが、ROCm のメインターゲットであるサーバー向け SKU が EOL に入った以上、同様にサポートから外されることが考えられる。  

[^mi25-eol]: [RadeonOpenCompute/ROCm at f088317e4483b8f75ec55ac3bb040f23e5abd2d4](https://github.com/RadeonOpenCompute/ROCm/tree/f088317e4483b8f75ec55ac3bb040f23e5abd2d4#Deprecations)

今リリースでサポートに追加された AMD GPU は無いが、[ROCm Installation Guide](https://rocmdocs.amd.com/en/latest/Installation_Guide/Installation_new.html) では対応リストに *Sienna Cichlid/Navi21 (gfx1030)* ベースの **Radeon Pro W6800** がさり気なく追加されており、ROCmソフトウェア/ライブラリも *gfx1030* に対応している。  

 * [ROCm Installation Guide v4.5 — ROCm Documentation 1.0.0 documentation](https://rocmdocs.amd.com/en/latest/Installation_Guide/Installation_new.html#confirm-you-have-a-rocm-capable-gpu)

余談に近いが、**Radeon Pro W6800** には `DeviceID: 0x73A3` が割り当てられている。だが [Tensile](https://github.com/ROCmSoftwarePlatform/Tensile) 等ではまた別の *Navi21/Sienna Cichlid* の `DeviceID: 0x73A2`、まだリリースされていない SKU を対象としている。[^73a2h]  
これが間違いでなければ、**Radeon Pro W6800** 以外に *Navi21/Sienna Cichlid* ベースのワークステーション向け、あるいはサーバー向けの SKU を計画しているのかもしれない。  

[^73a2h]: [AMD Radeon™ Pro V520 Graphics | AMD](https://www.amd.com/en/products/server-accelerators/amd-radeon-pro-v520#product-specs)

以前には *RDNA/GFX10* 系 GPU のサポートを 2021年に追加する予定があると述べられており、それが一応達成されたことにはなるが、[ROCmSupport](https://github.com/ROCmSupport) アカウントは ROCm v5.0 で *RDNA/GFX10* 系の公式サポートについて良いニュースがあるかもしれない、とコメントしている。  

 * <https://github.com/RadeonOpenCompute/ROCm/issues/1180#issuecomment-942101147>

**ROCm Installation Guide** に **Radeon Pro W6800** が載ってるのはミスで、正式なサポートは ROCm v5.0 で追加するつもりなのだろうか。  
ただ他 *RDNA 2/GFX10.3* 世代の GPU がサポートされるか、*RDNA 1/GFX10.1* 世代もサポートされるかは不透明なまま。  

{{< ref >}}
 * [AMD Radeon™ Pro V520 Graphics | AMD](https://www.amd.com/en/products/server-accelerators/amd-radeon-pro-v520#product-specs)
 * [Radeon Instinct™ MI25 Accelerator | Server Accelerators| AMD](https://www.amd.com/en/products/professional-graphics/instinct-mi25#product-specs)
{{< /ref >}}
