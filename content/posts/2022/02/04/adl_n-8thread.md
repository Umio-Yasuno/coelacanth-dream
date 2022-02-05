---
title: "CPU 8-Thread、GPU 32EU を持つ Alder Lake-N"
date: 2022-02-04T15:14:48+09:00
draft: false
tags: [ "Alder_Lake", ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "CPU" ]
noindex: false
# summary: ""
---

Intel の [Vamshigopal](https://github.com/Vamshigopal) 氏より、Sound Open Firmware (SOF) Project における Linuxレポジトリに *Alder Lake-N (ADL-N RVP)* の一部ブートログがアップロードされていた。  
バグ報告のためアップロードされたログであり、得られる情報は少ないが、これまでに集めた情報と照らし合わせることはできる。  

* [IO transfer error timeout and system suspend-resume test fails · Issue #3401 · thesofproject/linux](https://github.com/thesofproject/linux/issues/3401#issuecomment-1028638063)

## CPU (Model: 0xBE) {#cpu}
当該の CPU はエンジニアサンプリングとされ、CPU Name は *Genuine Intel(R) 0000* となっている。  
CPUID Model は `0xBE (190)` であり、これは Coreboot の情報と一致し、確かに *Alder Lake-N* と言える。  
{{< link >}} [N バリアントも存在する Alder Lake | Coelacanth's Dream](/posts/2021/11/16/coreboot-intel-adl_n/#cpuid) {{< /link >}}
ただ、*Alder Lake* の CPUID Model として、最近 `0xBF (191)` 追加されたが、*Alder Lake-N* が `0xBE (190)` で確定したとなると、単なる誤字である可能性が濃厚になった。あるいはまた別のバリアントが用意されている可能性もあるが。  
{{< link >}} [Intel Alder Lake と Rocket Lake に追加される CPUID Model | Coelacanth's Dream](/posts/2022/01/07/intel-adl-rkl-new-model/#adl) {{< /link >}}
GCC には `0xBF (191)` を追加するパッチが既に公開されているが、`0xBE (190)` については現在 Coreboot のみサポートが進められており、Linux Kernel はまだパッチが公開されていない。  
*Alder Lake-N* の GPU部、I/O部の DeviceID を Linux Kernel に追加するパッチは投稿、公開されている。  

 > 		[    0.345485] smpboot: CPU0: Genuine Intel(R) 0000 (family: 0x6, model: 0xbe, stepping: 0x0)
 >
 > {{< quote >}} [IO transfer error timeout and system suspend-resume test fails · Issue #3401 · thesofproject/linux](https://github.com/thesofproject/linux/issues/3401#issuecomment-1028638063) {{< /quote >}}

CPU Thread(s) は 8-Threads が認識されている。  
*Alder Lake-N* は Coreboot の関連コード部から、他 *Alder Lake* バリアントのようにハイブリッド構成を採らず、Atom系コアのみで構成すると考えられる。  
{{< link >}} [Alder Lake-N は Atom系コアのみの構成か | Coelacanth's Dream](/posts/2021/11/25/adl_n-atom-only/) {{< /link >}}
コアあたりのスレッド数も 1-Thread で変わらないとすると、*Alder Lake-N* は *Gracemont* 8-Core/8-Thread、4-Core で L2キャッシュを共有するクラスタを 2基を持つ。  
現世代のモバイル向け Atom系プロセッサ *Jasper Lake (Tremont)* は最大 4-Core (1-Cluster) 構成であったため、アーキテクチャの拡張も含めれば CPU部の規模は倍以上となる。  

 > 		[    0.376494] smp: Brought up 1 node, 8 CPUs
 > 		[    0.377487] smpboot: Max logical packages: 2
 > 		[    0.378486] smpboot: Total of 8 processors activated (19009.53 BogoMIPS)
 >
 > {{< quote >}} [IO transfer error timeout and system suspend-resume test fails · Issue #3401 · thesofproject/linux](https://github.com/thesofproject/linux/issues/3401#issuecomment-1028638063) {{< /quote >}}

## Gen12 GT1 32EU {#gpu}
GPU部の DeviceID は `0x46D0` 、これは *Alder Lake-N* の DeviceID に割り当てられており、ここでも *Alder Lake-N* であることを確かめられる。[^adl_n-did]  
各 IP の世代は、Graphics: Gen12、Media: Gen12、Display: Gen13 (Xe_LPD) であり、これはモバイル向けバリアントである *Alder Lake-P/M (Model: 0x9A)* と同じ。  

[^adl_n-did]: [[Intel-gfx] [PATCH V3] drm/i915/adl-n: Enable ADL-N platform](https://lists.freedesktop.org/archives/intel-gfx/2021-December/285043.html)

 > 		[    2.276814] i915 device info: pciid=0x46d0 rev=0x00 platform=ALDERLAKE_P (subplatform=0x1) gen=12
 > 		[    2.280151] i915 device info: graphics version: 12
 > 		[    2.281986] i915 device info: media version: 12
 > 		[    2.283719] i915 device info: display version: 13
 > 		[    2.285521] i915 device info: gt: 0
 > 		[    2.286870] i915 device info: iommu: disabled
 > 		[    2.288544] i915 device info: memory-regions: 5
 > 		[    2.290291] i915 device info: page-sizes: 211000
 > 		[    2.292045] i915 device info: platform: ALDERLAKE_P
 >
 > {{< quote >}} [IO transfer error timeout and system suspend-resume test fails · Issue #3401 · thesofproject/linux](https://github.com/thesofproject/linux/issues/3401#issuecomment-1028638063) {{< /quote >}}

規模は 32EU (1 Slice, 2 Sub-Slice, 16 EU per Sub-Slice)。  
*Alder Lake-P/M* は最大 96EU だったため、デスクトップ向けと同様に GPU部の規模をだいぶ抑えている。  
*Jasper Lake* は Gen11 32EU であり、Atom系プロセッサらしい規模を保ったとも見える。CPU部は最大 8-Thread に増やした一方で、GPU部は 32EU から増やさなくてもいいという判断がなされたのだろう。  
ただ SDP (Scenario Design Power) 7W という省電力プロセッサだった *Lakefield* が Gen11 64EU を持っていたことを思うと、Atom系プロセッサが求められる性能、電力帯で CPU と GPU、どちらの性能がより求められているかは難しい部分だと感じられる。  
{{< link >}} [Alder Lake-N を搭載する Chromebookボード 「Nissa, Nivviks, Nereid」 | Coelacanth's Dream](/posts/2022/01/12/adl_n-chromebook-board/) {{< /link >}}

 > 		[    2.377090] i915 device info: slice total: 1, mask=0001
 > 		[    2.379108] i915 device info: subslice total: 2
 > 		[    2.380844] i915 device info: slice0: 2 subslices, mask=00000003
 > 		[    2.383121] i915 device info: EU total: 32
 > 		[    2.384684] i915 device info: EU per subslice: 16
 > 		[    2.386481] i915 device info: has slice power gating: yes
 > 		[    2.388535] i915 device info: has subslice power gating: no
 > 		[    2.390652] i915 device info: has EU power gating: no
 >
 > {{< quote >}} [IO transfer error timeout and system suspend-resume test fails · Issue #3401 · thesofproject/linux](https://github.com/thesofproject/linux/issues/3401#issuecomment-1028638063) {{< /quote >}}

| | Alder Lake-N |
| :-- | :--: |
| Model | 0xBE |
| CPU (big + small) | 0 + 8? |
| GPU (Gen12.2) | GT1 32EU |
| CPU PCIe | N/A |
| PCH PCIe | 9-Lane |
