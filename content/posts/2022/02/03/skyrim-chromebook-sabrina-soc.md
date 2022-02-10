---
title: "AMD Sabrina SoC を搭載する Chromebookボード 「Skyrim」"
date: 2022-02-03T20:43:32+09:00
draft: false
tags: [ "Coreboot", "Chromebook", "Sabrina" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
---

Google の Karthik Ramasubramanian 氏より、Coreboor に *AMD Sabrina SoC* を搭載する Chromebookボード *Skyrim* のサポートを追加する最初のパッチが投稿された。  
Karthik Ramasubramanian 氏は、*AMD Sabrina SoC* プラットフォームにおける LPDDR5メモリのサポートを追加するパッチも投稿している。  
{{< link >}} [AMD Sabrina SoC は LPDDR5メモリをサポート | Coelacanth's Dream](/posts/2022/02/02/amd-sabrina-lpddr5/) {{< /link >}}
*Skyrim* は世界的に有名なあのゲームから取っていると思われ、これまでにも AMD APU/SoC を搭載する Chromebookボードにはゲームから取ったコードネームが付けられている。  
{{< link >}} [Lucienne/Cezanne APU を搭載する Chromebookボード　―― Guybrush、Mancomb | Coelacanth's Dream](/posts/2020/11/22/lcn-czn-fp6-chromebook-board/) {{< /link >}}

* [mb/google/skyrim: Add new mainboard (I008fea4a) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/61565)

*Skyrim* のサポートは ChromiumOSレポジトリでも進められており、*Skyrim6w* と *Skyrim15w* のボード 2種が確認できる。  

* [Update with publically filtered configs. (I8dd377f3) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/project/+/3319443)

 > 		        "identity": {
 > 		          "platform-name": "Skyrim",
 > 		          "sku-id": 2147483647,
 > 		          "smbios-name-match": "Skyrim6w"
 > 		        },
 > 		        "name": "skyrim6w",
 >
 > {{< quote >}} […/project-config.json · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/project/+/3319443/3/skyrim/skyrim6w/sw_build_config/platform/chromeos-config/generated/project-config.json#24) {{< /quote >}}

 > 		        "identity": {
 > 		          "platform-name": "Skyrim",
 > 		          "sku-id": 2147483647,
 > 		          "smbios-name-match": "Skyrim15w"
 > 		        },
 > 		        "name": "skyrim15w",
 >
 > {{< quote >}} […/project-config.json · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/project/+/3319443/3/skyrim/skyrim15w/sw_build_config/platform/chromeos-config/generated/project-config.json) {{< /quote >}}

素直に *Skyrim* には TDP: 6W と 15W の構成が計画されていると捉えれば、*AMD Sabrina APU/SoC* は *Raven2 (Zen 2-Core/4-Thread + Vega 3-CU) APU/SoC* の後継と見ることができる。  
{{< link >}} [Raven2 | Coelacanth's Dream](/tags/raven2/) {{< /link >}}

*Raven2* は *Dali* と *Pollock* のバリアントが存在し、バリアント間の区別にはまずパッケージの違いが挙げられる。  
*Pollock* では *FT5 パッケージ* を採用し、DDR4 1ch、SATAインターフェイス無しという省電力、低コスト志向の仕様となっている。  
*PollocK* ベースの SKU には現在 *AMD 3015e/3015Ce* が存在し、TDP は 6W となっている。  
*AMD Sabrina* もまた、SATAコントローラを持たず、PCIeポート数が他 APU と比較して少ないなど、*Raven2* と同じ方向性を持っている。  
{{< link >}} [AMD Sabrina SoC に向けたさらなるパッチ | Coelacanth's Dream](/posts/2022/01/14/amd-sabrina-soc-more-patch/#pcie) {{< /link >}}
メモリインターフェイス部は LPDDR5メモリをサポートしているとされるため、DDR4メモリのみをサポートしていた *Raven2* よりも省電力かつ広帯域が期待できる。  
*AMD Sabrina* の CPU/GPUアーキテクチャはまだ明かされておらず、*Raven2* から更新されている可能性があり、そうした点でも *AMD Sabrina* には期待が持てる。  

{{< ref >}}
* [AMD 3015e | AMD](https://www.amd.com/en/product/10166)
* [AMD 3015Ce | AMD](https://www.amd.com/en/product/11201)
{{< /ref >}}
