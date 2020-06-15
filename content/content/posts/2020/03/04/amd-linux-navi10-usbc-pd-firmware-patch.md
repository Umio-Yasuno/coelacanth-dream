---
title: "Linux Kernel amd-gfxにUSB-C PDファームウェアのパッチが投稿される"
date: 2020-03-04T11:24:10+09:00
draft: false
tags: [ "Radeon", "Navi10" ]
keywords: [ "", ]
categories: [ "Software", "GPU", "AMD" ]
noindex: false
---

RDNA世代で初のUSB-Cコネクタを実装した、Radeon Pro W5700が出てから気付けばもう3ヶ月近くが経ったが、先日Linux Kernel amd-gfxにUSB-C PD (Power Deliver)のファームウェアに関するパッチが投稿された。  
[[PATCH v3 0/3] Add support for USB-C PD FW download](https://lists.freedesktop.org/archives/amd-gfx/2020-March/046843.html)  

パッチの内容としては、I2Cを経由してファームウェアをUSB-Cコントローラーに適用、コントロールファイルを作成しPSP (Platform Security Processor)で制御するというもの。  

以下のパッチを読むとASICはNavi10だけが対象とされており、  
[[PATCH v3 3/3] drm/amdgpu: Add support for USB-C PD FW download](https://lists.freedesktop.org/archives/amd-gfx/2020-March/046846.html)  
USB-Cコントローラーを内蔵しているのがNavi10のみ（Navi12, Navi14には無い）、  
もしくはコントローラーは内蔵されていても、コネクタが実装されるのは現状Navi10ベースの製品しか予定にない、ということだろう。  

Navi14ベースのRadeon Pro 5300M /5500Mを搭載した[MacBook Pro 16-inch](https://www.apple.com/macbook-pro-16/specs/)にもUSB-C (Thunderbolt 3)は実装されているが、  
そちらは Intel JHL7540 Thunderbolt 3コントローラーを経由するため、問題無いのだと思われる。[^1][^2]  

また、[linux-firmware.git](https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/log/)にはまだ当のUSB-C PDファームウェアはアップロードされていない。  

[^1]: <cite>[MacBook Pro 16" 2019 Teardown - iFixit](https://www.ifixit.com/Teardown/MacBook+Pro+16-Inch+2019+Teardown/128106#s249785)</cite>
[^2]: <cite>[Intel® JHL7540 Thunderbolt™ 3 Controller Product Specifications](https://ark.intel.com/content/www/us/en/ark/products/97400/intel-jhl7540-thunderbolt-3-controller.html)</cite>

