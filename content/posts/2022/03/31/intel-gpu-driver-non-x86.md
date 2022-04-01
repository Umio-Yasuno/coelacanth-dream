---
title: "x86-64 CPU 以外のサポートを進める Intel GPUドライバー"
# date: 2022-03-31T03:11:33+09:00
date: 2022-03-31T04:11:33+09:00
draft: false
categories: [ "Software", "Intel", "GPU" ]
# tags: [ "", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Intel dGPU 単体の市場への投入が近付くにつれ、Intel GPUドライバーでは x86(-64) 以外の CPU を採用するプラットフォームへの対応が進められている。  

dGPU 単体としてはすでに Intel DG1 /Iris Xe MAX /SG1 ({{< xe class="lp" >}}) がいるが、それらはほとんどが限定したプラットフォームでのリリースであり、また専用の UEFI/BIOS をインストールしたシステム (CPU+M/B) を必要としていた。[^dg1-bios]
そのため当時は、x86(-64) 以外の対応は優先事項ではなかったと思われる。  
また専用の UEFI/BIOS を必要することについて、DG1 の PCI OpROM と VBIOS のイメージレイアウト、Intel の Subrata Banik 氏より Coreboot に投稿された、dGPUカード内部に埋め込まれた PCI OpROM への対応が関係しているのではないかと考えているが、公式にはっきりと説明されている訳ではなく、確証はない。[^coreboot-pci-oprom]  

[^dg1-bios]: [Intel Iris Xe Discrete Card Will Only Work With Select CPUs and Motherboards - Legit Reviews](https://www.legitreviews.com/intel-iris-xe-discrete-card-will-only-work-with-select-cpus-and-motherboards_225470)
[^coreboot-pci-oprom]: [device: Add new Kconfig VGA_ROM_RUN_DEFAULT for mainboard user (Iecb2fcdb) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/49016)

{{< pindex >}}
 * [対応における課題](#fence)
{{< /pindex >}}

 > 		+/**
 > 		+ *	+        DASH+G OPROM IMAGE LAYOUT           +
 > 		+ *	+--------+-------+---------------------------+
 > 		+ *	| Offset | Value |   ROM Header Fields       +-----> Image 1 (CSS)
 > 		+ *	+--------------------------------------------+
 > 		+ *	|    0h  |  55h  |   ROM Signature Byte1     |
 > 		+ *	|    1h  |  AAh  |   ROM Signature Byte2     |
 > 		+ *	|    2h  |  xx   |        Reserved           |
 > 		+ *	|  18+19h|  xx   |  Ptr to PCI DataStructure |
 > 		+ *	+----------------+---------------------------+
 > 		+ *	|           PCI Data Structure               |
 > 		+ *	+--------------------------------------------+
 > 		+ *	|    .       .             .                 |
 > 		+ *	|    .       .             .                 |
 > 		+ *	|    10  +  xx   +     Image Length          |
 > 		+ *	|    14  +  xx   +     Code Type             |
 > 		+ *	|    15  +  xx   +  Last Image Indicator     |
 > 		+ *	|    .       .             .                 |
 > 		+ *	+--------------------------------------------+
 > 		+ *	|               MEU BLOB                     |
 > 		+ *	+--------------------------------------------+
 > 		+ *	|              CPD Header                    |
 > 		+ *	|              CPD Entry                     |
 > 		+ *	|              Reserved                      |
 > 		+ *	|           SignedDataPart1                  |
 > 		+ *	|              PublicKey                     |
 > 		+ *	|            RSA Signature                   |
 > 		+ *	|           SignedDataPart2                  |
 > 		+ *	|            IFWI Metadata                   |
 > 		+ *	+--------+-------+---------------------------+
 > 		+ *	|    .   |   .   |         .                 |
 > 		+ *	|    .   |   .   |         .                 |
 > 		+ *	+--------------------------------------------+
 > 		+ *	| Offset | Value |   ROM Header Fields       +-----> Image 2 (Config Data) (Offset: 0x800)
 > 		+ *	+--------------------------------------------+
 > 		+ *	|    0h  |  55h  |   ROM Signature Byte1     |
 > 		+ *	|    1h  |  AAh  |   ROM Signature Byte2     |
 > 		+ *	|    2h  |  xx   |        Reserved           |
 > 		+ *	|  18+19h|  xx   |  Ptr to PCI DataStructure |
 > 		+ *	+----------------+---------------------------+
 > 		+ *	|           PCI Data Structure               |
 > 		+ *	+--------------------------------------------+
 > 		+ *	|    .       .             .                 |
 > 		+ *	|    .       .             .                 |
 > 		+ *	|    10  +  xx   +     Image Length          |
 > 		+ *	|    14  +  xx   +      Code Type            |
 > 		+ *	|    15  +  xx   +   Last Image Indicator    |
 > 		+ *	|    .       .             .                 |
 > 		+ *	|    1A  +  3C   + Ptr to Opregion Signature |
 > 		+ *	|    .       .             .                 |
 > 		+ *	|    .       .             .                 |
 > 		+ *	|   83Ch + IntelGraphicsMem                  | <---+ Opregion Signature
 > 		+ *	+--------+-----------------------------------+
 > 		+ *
 > 		+ * intel_oprom_verify_signature() verify OPROM signature.
 > 		+ * @opreg: pointer to opregion buffer output.
 > 		+ * @opreg_size: pointer to opregion size output.
 > 		+ * @dev_priv: i915 device.
 > 		+ */
 >
 > {{< quote >}} [[Intel-gfx] [RFC PATCH 120/162] drm/i915/oprom: Basic sanitization](https://lists.freedesktop.org/archives/intel-gfx/2020-November/254123.html) {{< /quote >}}

## 対応における課題 {#fence}

現時点で、x86(-64) 以外のプラットフォームへのサポートを進める Intel GPUドライバーと関連ライブラリには、Mesa3D、IGC (Intel Graphics Compiler)、gmmlib (Graphics Memory Management Library)、Compute Runtime が確認できる。  
サポートに追加される対象は主に ARM64 (aarch64) となっている。  

 * [intel: Support compiling on non-x86 (!15309) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/15309)
 * [Introduce ARM64 Support for the Library (#91) · intel/gmmlib@32f4cfc](https://github.com/intel/gmmlib/commit/32f4cfc294f8342f6e0e1bae2ef430c3971d50c6)
 * [Enable ARM build · intel/intel-graphics-compiler@eda4042](https://github.com/intel/intel-graphics-compiler/commit/eda4042be98cf1edd6cabf45104f4e1050a3bbfb)
 * [Add neon intrinsics for aarch64 · intel/compute-runtime@cf90603](https://github.com/intel/compute-runtime/commit/cf906030ac0da5799d6a8ee7878f64bb20fa8b9c)

まだ正式に対応してないため作業途中 (WIP) と思われるが、内容としては主に x86(-64) 固有の命令を直接使っている部分を、複数のアーキテクチャに対応したライブラリに置き換えるか、マクロで制御しコードを分ける、といった対応を取っている。  
対応が必要な命令として挙げられているものには、主に `SSE/2/3/4.1/4.2`,`CLFLUSH`, `CPUID` がある。  
`SSE` は SIMD命令として CPU側の計算や変換処理、`CLFLUSH` はキャッシュラインをメモリにフラッシュし、同期を取るため、`CPUID` は CPU の拡張命令、機能の対応フラグの取得に使われている。  

gmmlib では [sse2neon](https://github.com/DLTcollab/sse2neon/) を導入することで SSE系命令を Arm NEON 命令に置き換えている。  
Mesa3D では、一度最低限の対応を行うマージリクエストが開設されたが、将来的に non x86(-64) + Intel dGPU というプラットフォームが出てくる可能性は非常に高いとしつつも、まだ Intel dGPU、Intel Arc Aシリーズはリリースされていないため、今は閉じられている。  
また、私見ではあるが、PCIeスロットが実装された non x86(-64) プラットフォームには Ampere eMAG (ARM64/aarch64)[^ampere-emag] や Raptor Talos II (POWER9) [^raptor-talos-ii] 存在するが、一般的に普及しているものではない。  
また、Intel Arc Aシリーズのリリース後もしばらくはそちらの対応にリソースを割きたいという考えもドライバー開発者にはあるのではないかと思われる。  

[^ampere-emag]: [Conclusion - All Eyes on an Altra System - Avantek's Arm Workstation: Ampere eMAG 8180 32-core Arm64 Review](https://www.anandtech.com/show/15733/ampere-emag-system-a-32core-arm64-workstation/7)
[^raptor-talos-ii]: [Raptor Computing Systems::Talos™ II Secure Workstation](https://www.raptorcs.com/TALOSII/)

{{< ref >}}
 * DG1
    * [[Intel-gfx] [PATCH 0/2] Splitting intel-gtt calls for non-x86 platforms](https://lists.freedesktop.org/archives/intel-gfx/2022-March/293151.html)
        * [[Intel-gfx] [PATCH 1/2] drm/i915: Require INTEL_GTT to depend on X86](https://lists.freedesktop.org/archives/intel-gfx/2022-March/293152.html)
        * [[Intel-gfx] [PATCH 2/2] drm/i915/gt: Split intel-gtt functions by arch](https://lists.freedesktop.org/archives/intel-gfx/2022-March/293153.html)
    * [Intel Iris Xe Discrete Card Will Only Work With Select CPUs and Motherboards - Legit Reviews](https://www.legitreviews.com/intel-iris-xe-discrete-card-will-only-work-with-select-cpus-and-motherboards_225470)
{{< /ref >}}
