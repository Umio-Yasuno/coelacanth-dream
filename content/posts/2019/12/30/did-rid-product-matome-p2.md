---
title: "AMDGPU Database: Device ID/ Revision ID/ Product Name"
date: 2019-12-30T10:59:25+09:00
draft: false
tags: [ "Radeon" ]
keywords: [ "Radeon", "AMDGPU" , "Device ID", "Revision ID", "Product Name" ]
categories: [ "Hardware", "AMD", "GPU", "Database" ]
summary: " "
---

<p></p>

{{< pindex >}}

 * [APU](#apu)
    * FAMILY_RV
        * [Raven](#raven-gfx902)
        * [Picasso](#picasso-gfx902)
        * [Raven2](#raven2-gfx909)
            * [Dali](#dali-gfx909)
            * [Pollock](#pollock-gfx909)
        * [Renoir](#renoir-gfx90c)
        * [Lucienne](#lucienne-gfx90c)
        * [Cezanne](#cezanne-gfx90c)
        * [Barcelo](#barcelo)
    * FAMILY_VGH
        * [VanGogh](#vgh)
    * FAMILY_YC
        * [Yellow Carp](#yc)
 * [Discrete GPU](#dgpu)
    * FAMILY_AI
        * [Vega10](#vega10-gfx900)
        * [Vega12](#vega12-gfx904)
        * [Vega20](#vega20-gfx906)
        * [Arcturus](#arcturus-gfx908)
        * [Aldebaran](#aldebaran-gfx90a)
    * FAMILY_NV
        * [Navi10](#navi10-gfx1010)
        * [Navi12](#navi12-gfx1011)
        * [Navi14](#navi14-gfx1012)
        * [Sienna Cichlid/Navi21](#sienna_cichlid-gfx1030)
        * [Navy Flounder/Navi22](#navy_flounder-gfx1031)
        * [Dimgrey Cavefish/Navi23](#dimgrey_cavefish-gfx1032)
        * [Beige Goby/Navi24](#beige_goby-gfx1034)
 * [参考リンク](#reference_title)

{{< /pindex >}}

## APU {#apu}
### FAMILY_RV {#family_rv}
#### Raven ( gfx902 ) {#raven-gfx902}
| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 0x15DD &darr; | 0x81 | V1807B | (45W FP5 Vega 11)
|  | 0x82 | V1756B | (45W FP5 Vega 8)
| | 0x83 | V1605B | (15W FP5 Vega 8)
| | 0x84 | iTemp EMBEDDED | (15W FP5 Vega 6)
| | 0x85 | V1202B | (15W FP5 Vega 3)
| | 0x86 |  | (Embedded 65W AM4)
| | 0x87 |  | (Embedded 65W AM4)
| | 0x88 |  | (Embedded 65W AM4 Vega 8)
| | 0xC1 | 2800H? | (45W FP5 Vega 11) |
| | 0xC2 | 2600H? | (45W FP5 Vega 8) |
| | 0xC3 | 2700U | (15W FP5 Vega 10) |
| | 0xC4 | 2500U | (15W FP5 Vega 8) |
| | 0xC5 | 2200U | (15W FP5 Vega 3) |
| | 0xC6 | 2400G | (65W AM4 Vega 11) |
| | 0xC7 | | (B8 65W AM4) |
| | 0xC8 | 2200G | (65W AM4 Vega 8) |
| | 0xC9 | 2400GE[^rv-2400ge] | (35W AM4 Vega 11) |
| | 0xCA | 2200GE? | (35W AM4 Vega 8) |
| | 0xCB | 200GE /220GE /240GE | (35W AM4 Vega 3) |
| | 0xCC | 2300U | (15W FP5 Vega 6) |
| | 0xCE |  | (15W FP5 Vega 3) |
| | 0xD0 | PRO 2700U[^rv-pro-2700u] | (15W FP5 Vega 10) |
| | 0xD1 | PRO 2500U[^rv-pro-2500u] | (15W FP5 Vega 8) |
| | 0xD2 | | 0x(B4 15W FP5) |
| | 0xD3 | PRO 2400G[^rv-pro-2400g] | (65W AM4 Vega 11) |
| | 0xD4 | | 0x(B8 65W AM4) |
| | 0xD5 | PRO 2200G? | (65W AM4 Vega 8) |
| | 0xD6 | PRO 2400GE[^13] | (35W AM4 Vega 11) |
| | 0xD7 | PRO 2200GE[^rv-pro-2200ge] | (35W AM4 Vega 8) |
| | 0xD8 | | (35W AM4 Vega 3) |
| | 0xD9 | PRO 2300U[^rv-pro-2300u] | (15W FP5 Vega 6) |
||
| | ? | V1404I | (15W FP5 Vega 8) |

[Page Top](#page_index)

[^rv-pro-2300u]: <https://bugs.freedesktop.org/attachment.cgi?id=144502>
[^rv-pro-2700u]: [LKML: Ignat Insarov: Non-deterministically boot into dark screen with \`amdgpu\`](https://lkml.org/lkml/2020/8/9/70)
[^13]: [207899 – AMDGPU large block noise in chrome, on streaming video pages. CS:GO freezes.](https://bugzilla.kernel.org/show_bug.cgi?id=207899)
[^rv-pro-2200ge]: [AMDGPU Linux Driver Preparing To Better Support Modern HDR/OLED Displays - Phoronix Forums](https://www.phoronix.com/forums/forum/linux-graphics-x-org-drivers/open-source-amd-linux/1158423-amdgpu-linux-driver-preparing-to-better-support-modern-hdr-oled-displays)
[^rv-pro-2400g]: [Question #692387 : Questions : Ubuntu](https://answers.launchpad.net/ubuntu/+question/692387)
[^rv-2400ge]: <https://forum.openmandriva.org/uploads/default/original/2X/5/5b23ce7e839de8a27f84040eb6a779afe7e19fe7.txt>
[^rv-pro-2500u]: [openbsd dev - bugs - Hibernate/Suspend not working with AMD Ryzen 5](http://openbsd-archive.7691.n7.nabble.com/Hibernate-Suspend-not-working-with-AMD-Ryzen-5-td367226.html)

#### Picasso ( gfx902 ) {#picasso-gfx902}
| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 0x15D8 &darr; | 00 | | Winston (Vega 8 WS) |
| | 0x93 | | (Vega 1 Raven2? /Dali? /Pollock?) |
| | 0xA1 | | (Vega 10) |
| | 0xA2 | | (Vega 8) |
| | 0xA3 | | (Vega 6) |
| | 0xA4 | | (Vega 3) |
| | 0xB1 | | (Vega 10) |
| | 0xB2 | | (Vega 8) |
| | 0xB3 | | (Vega 6) |
| | 0xB4 | | (Vega 3) |
| | 0xC1 | 3700C/U[^7] /3750H | (Vega 10) |
| | 0xC2 | 3500C/U[^7] /3550H | (Vega 8) |
| | 0xC3 | | (Vega 6) |
| | 0xC4 | &rarr; [Raven2](#raven2-gfx909) 3200U | |
| | 0xC5 | &rarr; [Raven2](#raven2-gfx909) 300U | |
| | 0xC8 | 3350G[^pco-3350g]/3400G | (AM4 Vega 11) |
| | 0xC9 | 3200G | (AM4 Vega 8) |
| | 0xCA | | (AM4 Vega 11) |
| | 0xCB | | (AM4 Vega 8) |
| | 0xCC | &rarr; [Raven2](#raven2-gfx909) 300GE /3000G | |
| | 0xCD | | (AM4 Vega 2) |
| | 0xD1 | PRO 3700U | (Vega 10) |
| | 0xD2 | PRO 3500U | (Vega 8) |
| | 0xD3 | (PRO) 3300U | (Vega 6) |
| | 0xD4 | | (Vega 3) |
| | 0xD8 | PRO 3400G[^pco-pro-3400g] | (AM4 Vega 11) |
| | 0xD9 | PRO 3200G[^pco-pro-3200g] | (AM4 Vega 8) |
| | 0xDA | PRO 3400GE[^pco-pro-3400ge] | (AM4 Vega 11) |
| | 0xDB | PRO 3200GE[^pco-pro-3200ge] | (AM4 Vega 8) |
| | 0xDC | PRO 300/320GE?? | (AM4 Vega 3) |
| | 0xE1 | 3780U | Winston (Surface Edition Vega 11) |
| | 0xE2 | 3580U | Winston (Surface Edition Vega 8) |
||
| | ? | 300UGE | (65W AM4 Vega 3) |
| | ? | 3150G | (65W AM4 Vega 3) |
| | ? | 3150GE | (35W AM4 Vega 3) |
| | ? | PRO 3125GE | (35W AM4 Vega 3) |
| | ? | PRO 3150G | (65W AM4 Vega 3) |
| | ? | PRO 3150GE | (35W AM4 Vega 3) |
| | ? | PRO 3350G | (65W AM4 Vega 10) |
| | ? | PRO 3350GE | (35W AM4 Vega 10) |
| | ? | 3450U | |

[Page Top](#page_index)

[^pco-pro-3400ge]: [PassMark Software - Display Baseline ID# 1308142](https://www.passmark.com/baselines/V8/display.php?id=130814241228)
[^pco-pro-3200ge]: [PassMark Software - Display Baseline ID# 1300484](https://www.passmark.com/baselines/V8/display.php?id=130048431252)
[^pco-pro-3400g]: <http://www.ozone3d.net/gpudb/score.php?which=900474>
[^pco-pro-3200g]: <https://gpuscore.top/msi/kombustor/show.php?id=505043>
[^pco-3350g]: [AMD Ryzen 5 3350G - lspci - OpenBenchmarking.org](https://openbenchmarking.org/system/2103299-HA-RYZEN335050/AMD%20Ryzen%205%203350G/lspci)

#### Raven2 ( gfx909 ) {#raven2-gfx909}
| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 0x15DD &darr; | 0xE1 | | (15W FP5 Vega 3) |
| | 0xE2 | | (35W AM4 Vega 3) |
| 0x15D8 &darr; | 0x91[^10] | R1606G | Embedded |
| | 0x92 | R1505G | Embedded |
| | 0xC4 | 3200U[^rv2-3200u] | (FP5 15W Vega 3), == 3250C? |
| | 0xC5 | 300U[^rv2-300u] | (Vega 3) |
| | 0xCC | 300GE[^rv2-300ge] /3000G[^rv2-3000g] | (AM4 Vega 3) |

[Page Top](#page_index)

[^rv2-3200u]: [FS#65359 : Wireless AC 3168 fails to initialize](https://bugs.archlinux.org/task/65359)
[^rv2-300u]: [HP Laptop 15-db1001nt DxDiag - Technopat Sosyal](https://www.technopat.net/sosyal/konu/hp-laptop-15-db1001nt-dxdiag.769397/)
[^rv2-3000g]: [LKML: A L: Re: AMDGPU crash on 5.4.7 on AMD Athlon 3000G APU](https://lkml.org/lkml/2020/1/3/224)
[^10]: [Core i7並みのRyzen搭載で、4万円台＆片手サイズ⁉ 〝Ryzen Embedded”搭載の超小型PCベアボーン「4×4 BOX」が超お得 (3/3) | AMD HEROES](https://amd-heroes.jp/article/2020/03/0364/3/)
[^rv2-300ge]: [PassMark Software - Display Baseline ID# 1314532](https://www.passmark.com/baselines/V8/display.php?id=131453210324)

##### Dali ( gfx909 ) {#dali-gfx909}
| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 0x15D8 &darr; | 0xC4 | 3250C[^7] | (Vega 3) |
| | 0xCD | Silver 3050C[^7] | (Vega 2) |
| | 0xCE | Gold 3150C[^7]/3150U[^gold-3150u] | (Vega 3) |
| | 0xCF | R1305G | (6W[^8]) |
| | 0xDE[^9] | | |
| | 0xDF[^9] | | |
| | 0xE3[^9] | | (6W[^8] Vega 3) |
| | 0xE4[^9] | R1102G[^11] | (6W[^8] Vega 3) |
||
| | ? | 3020e | |
| | ? | Silver 3050e | |
| | ? | 3050GE | (35W AM4 Vega 3) |

[Page Top](#page_index)

[^gold-3150u]: [bigining - dmesg - OpenBenchmarking.org](https://openbenchmarking.org/system/2109268-IB-FIRST907876/bigining/dmesg)
[^7]: [2040455: Rework map_oprom_vendev to add revision check and mapping —   Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2040455/3/src/soc/amd/picasso/northbridge.c#332)
[^8]: [[Dali] Raven 2 detection Patch](https://lists.freedesktop.org/archives/amd-gfx/2020-February/045579.html) <br> <https://lists.freedesktop.org/archives/amd-gfx/attachments/20200205/f22ebc46/attachment.obj>
[^9]: [[PATCH 28/35] drm/amd/display: Fix RV2 Variant Detection](https://lists.freedesktop.org/archives/amd-gfx/2020-February/046322.html)
[^11]: amdgpu.ids [R-Series R1000 with Radeon™ Vega Graphics Drivers & Support | AMD](https://www.amd.com/en/support/embedded/amd-ryzen-embedded-r-series-processors/r-series-r1000-radeon-vega-graphics)

##### Pollock ( gfx909 ) {#pollock-gfx909}
| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 0x15D8 &darr; | 0x94 | | |
| | 0x95 | | |
| | 0xE9 | | (Samples FT5)[^7] |
| | 0xEA | | (Production FT5)[^7] |
| | 0xEB | | |
| | | 3015e | |

[Page Top](#page_index)

 > Source:<cite>
[drm/amd/display: add Pollock IDs, fix Pollock & Dali clk mgr construct - amd-staging-drm-next](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd/display/include/dal_asic_id.h?h=amd-staging-drm-next&id=04d8707ef5549065c10d5a2e20d2c61089807997)</cite>

#### ?

| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 0x15D9 &darr; | 0x91 | | **?** |
| | 0x92 | | |
| | 0xC1 | | |
| | 0xC2 | | |
| | 0xC3 | | |

<!--
#### FireFlight
| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 15FF | 00 | Fenghuang (Zhongshan Subor Z+) | Radeon Vega 28 Mobile |

DG02SRTBP4MFA
-->

### Renoir ( gfx90c ) {#renoir-gfx90c}
| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 0x15E7[^rn-15e7h] | | | |
| 0x1636 &darr; | 0x00 | | (BringUp FP6) |
| | 0xC1 | Extreme Edition /4800U[^rn-4800u] | (B12 15W FP6) |
| | 0xC2 | 4700U[^rn-4700u] | (B10 15W FP6) |
| | 0xC3 | 4500U[^rn-4500u] | (B8 15W FP6) |
| | 0xC4 | 4300U[^rn-4300u] | (B6 15W FP6) |
| | 0xC5 | 4900HS[^rn-4900hs] | (B12 45W FP6) |
| | 0xC6 | 4800H[^rn-4800h]/HS[^rn-4800hs] | (B10 45W FP6) |
| | 0xC7 | 4600H[^rn-4600h] | (B8 45W FP6) |
| | 0xC8 | | (B10 65W AM4) |
| | 0xC9 | | (B8 65W AM4) |
| | 0xCA | | (B6 65W AM4) |
| | 0xCB | | (B10 35W AM4) |
| | 0xCC | | (B8 35W AM4) |
| | 0xCD | | (B6 35W AM4) |
| | 0xCE | 4600U?[^rn-4600u] | (B4 35W AM4) |
| | 0xD1 | PRO 4750U[^rn-pro-4750u] | (B12B 15W FP6) |
| | 0xD2 | | (B10B 15W FP6) |
| | 0xD3 | PRO 4650U[^rn-pro-4650u] | (B8B 15W FP6) |
| | 0xD4 | PRO 4450U[^rn-pro-4450u] | (B6B 15W FP6) |
| | 0xD5 | | (B12B 45W FP6) |
| | 0xD6 | | (B10B 45W FP6) |
| | 0xD7 | | (B8B 45W AM4) |
| | 0xD8 | PRO 4750G[^pro-desktop-65w-rn] | (B10B 65W AM4) |
| | 0xD9 | PRO 4650G[^pro-desktop-65w-rn] | (B8B 65W AM4) |
| | 0xDA | PRO 4350G[^pro-desktop-65w-rn] | (B6B 65W AM4) |
| | 0xDB | | (B10B 35W AM4) |
| | 0xDC | | (B8B 35W AM4) |
| | 0xDD | | (B6B 35W AM4) |
| | 0xDE | | (B4B 35W AM4) |
| | 0xE1 | | Acton |
| | 0xE2 | | Acton |
| | 0xE3 | | Acton |
| | 0xF0 | 4900H[^rn-4900h] | |
||
| | ? | 4700G |
| | ? | 4600G |
| | ? | 4300G |
| | ? | 4700GE |
| | ? | 4600GE |
| | ? | 4300GE |
| | ? | PRO 4750GE |
| | ? | PRO 4650GE |
| | ? | PRO 4350GE |

[^rn-15e7h]: [[PATCH] drm/amdgpu: add another Renior DID](https://lists.freedesktop.org/archives/amd-gfx/2021-July/066502.html)
[^rn-pro-4650u]: [Panic on boot with Renoir when testing drm-live-devel-20200906.img · Issue #23 · freebsd/drm-kmod](https://github.com/freebsd/drm-kmod/issues/23)
[^rn-4800u]: [New Ryzen 4800U Laptop with Radeon RX 5500M kernel freezes / Kernel & Hardware / Arch Linux Forums](https://bbs.archlinux.org/viewtopic.php?id=257345)
[^rn-4700u]: <https://www.mail-archive.com/tech@openbsd.org/msg59151.html>
[^rn-4900hs]: [Folding Forum • View topic - Ryzen 4900HS Renoir iGPU Not Receiving Work](https://foldingforum.org/viewtopic.php?f=108&t=35623)
[^rn-4500u]: [[Solved] strange BackLight problom on Acer swift 314-42(AMD 4500U) / Laptop Issues / Arch Linux Forums](https://bbs.archlinux.org/viewtopic.php?id=254614)
[^rn-4300u]: [Review IdeaPad Slim 3 – AMD Ryzen 3 4300U | Jagat Review](http://www.jagatreview.com/2020/07/review-ideapad-slim-3-amd-ryzen-3-4300u/)
[^rn-4600u]: [No second monitor (#31) · Issues · Xfce / exo · GitLab](https://gitlab.xfce.org/xfce/exo/-/issues/31)
[^rn-4800h]: [Suspend/resume issue for RX5600M/4800H (Dell G5 SE) (#1222) · Issues · drm / amd · GitLab](https://gitlab.freedesktop.org/drm/amd/-/issues/1222)
[^rn-4800hs]: [ASUS Zephyrus G GA401IV DxDiag - Technopat Sosyal](https://www.technopat.net/sosyal/konu/asus-zephyrus-g-ga401iv-dxdiag.804989/)
[^rn-4600h]: [Dell G5 15 SE SMU crash after switching from iGPU to dGPU (#1269) · Issues · drm / amd · GitLab](https://gitlab.freedesktop.org/drm/amd/-/issues/1269)
[^rn-4900h]: [Screen flickering with ASUS TUF GAMING A15 laptop (FX506IV) (#1271) · Issues · drm / amd · GitLab](https://gitlab.freedesktop.org/drm/amd/-/issues/1271)
[^pro-desktop-65w-rn]: [ASCII.jp：Renoirのデスクトップ版「Ryzen PRO 4000Gシリーズ」3モデルの性能を検証 (4/5)](https://ascii.jp/elem/000/004/021/4021569/4/)
[^rn-pro-4450u]: [ThinkPad L14 Ryzen 3 Pro - lspci - OpenBenchmarking.org](https://openbenchmarking.org/system/2010191-NE-TPL14GPUT90/ThinkPad%20L14%20Ryzen%203%20Pro/lspci)
[^rn-pro-4750u]: [Problem with GPU drivers on ThinkPad L15 Gen 1 (AMD) / Laptop Issues / Arch Linux Forums](https://bbs.archlinux.org/viewtopic.php?id=259892)

<!--  Lucienne (gfx90c?)  -->
### Lucienne ( gfx90c ) {#lucienne-gfx90c}
| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 0x164C &darr; | | | |
| | 0xC2 | 5500U[^lcn-5500u] | |

[^lcn-5500u]: <https://openbenchmarking.org/system/2105266-IB-5500USYNT82/1/lspci>

<!--  Cezanne (gfx90c?) -->
### Cezanne ( gfx90c ) {#cezanne-gfx90c}
| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 0x1638 &darr; | | | |
| | 0xC2 | 5600U[^czn-5600u] | |

[^czn-5600u]: [LinuxHardware/NoteBooks/Lenovo/ThinkPad_L14_Gen2_AMD - LinuxWiki.org - Linux Wiki und Freie Software](https://www.linuxwiki.de/LinuxHardware/NoteBooks/Lenovo/ThinkPad_L14_Gen2_AMD)

<!--  Barcelo (gfx90c?) -->
### Barcelo ( gfx90c ) {#barcelo}
| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 0x15E7[^barcelo-did] &darr; | | | |

[^barcelo-did]: [soc/amd/common/block/graphics: add GPU PCI ID for Barcelo (I1c5446c1) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/56396)


<!--
   Board:
      Celadon: AMD Mobile Reference Board?
         > AMD Celadon reference board (AMD Ryzen™ 5 4500U)
         <https://www.amd.com/en/processors/laptop-processors-for-business>
      Myrtle: AMD AM4 Platform code-name
         <https://drivers.amd.com/relnotes/amd_nvme_sata_raid_quick_start_guide_for_windows_operating_systems.pdf>
      Artic: Desktop? (PRO565?)
-->

### FAMILY_VGH {#family_vgh}
#### VanGogh {#vgh}
| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 0x163F &darr; | | | |

### FAMILY_YC {#family_yc}
#### Yellow Carp {#yc}
| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 0x164D[^yc-did] &darr; | | | |
| 0x1681 &darr; | 0xC7 | (AMD Eng Sample: 100-000000560-40_Y) | LilacKD-RMB[^lilackd-rmb] |

[^lilackd-rmb]: [[Kernel-packages] [Bug 1953008] ProcInterrupts.txt](https://www.mail-archive.com/kernel-packages@lists.launchpad.net/msg466205.html)

[^yc-did]: [[PATCH 2/2] drm/amdgpu: add yellow carp pci id (v2)](https://lists.freedesktop.org/archives/amd-gfx/2021-July/066869.html)

[Page Top](#page_index)

## Discrete GPU {#dgpu}
### FAMILY_AI {#family_ai}
#### Vega10 ( gfx900 ) {#vega10-gfx900}
| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 0x6860 &darr; | 0x00 | Instinct MI25 &darr; | (Vega10 GLXT SERVER) &darr; |
| | 0x01 | | |
| | 0x02 | | |
| | 0x03 | Pro V340 | |
| | 0x04 | Instinct MI25x2 | |
| | 0x07 | Pro V320 (/V340?) [^6]| |
| | 0xC0 | Pro Vega 48 /56 /64 /64X[^1] | ? |
| 0x6861 | 0x00 | Pro WX 9100 | (Vega10 GLXT) |
| 0x6862 | 0x00 | Pro SSG | (Vega10 SSG) |
| 0x6863 | 0x00 | Vega Frontier Edition | (Vega10 GLXTX) |
| 0x6864 &darr; | 00 | | |
| | 0x03 | Pro V340 | (Vega10 GLXT SERVER) &darr; |
| | 0x04 | Instinct MI25x2 | |
| | 0x05 | Pro V340 | |
| 0x6867 | 00 | Pro Vega 56 | (Vega10 XLA) |
| 0x6868 | 00 | (TM) Pro WX 8200 | (Vega10 GLXL) |
| 0x6869 | 00 | | (Vega10 XGA) |
| 0x686A | 00 | | (Vega10 LEA) |
| 0x686B | 00 | Pro Vega 64X | (Vega10 XTXA) |
| 0x686C &darr; | 00 | Instinct MI25 MxGPU &darr; | (Vega10 GLXT SERVER VF) &darr; |
| | 0x01 | | |
| | 0x02 | | |
| | 0x03 | Pro V340 MxGPU | |
| | 0x04 | Instinct MI25x2 MxGPU | |
| | 0x05 | Pro V340 MxGPU | |
| | 0xC1 | | |
| 0x686D | 00 | | (Vega10 GLXTA) |
| 0x686E | 00 | | (Vega10 GLXLA) |
| 0x687F &darr; | 01 | RX Vega | (Vega10 XT - Blade) |
| | 0xC0 | RX Vega | (Vega10 XTX) |
| | 0xC1 | RX Vega (64) | (Vega10 DESKTOP XT) |
| | 0xC3 | RX Vega (56) | (Vega10 XL) |
| | 0xC4 | | (Vega10 XL) |
| | 0xC7 | RX Vega | (Vega10 XL) |

[Page Top](#page_index)

[^1]: SubSystem ID? ( Vega 48:0x0196, Vega 56:0x017B, Vega 64:0x017C Vega 64X:0x0188 )
[^6]: [Compute Performance of AMD Radeon Pro V320 - CompuBench](https://compubench.com/device.jsp?benchmark=compu20d&os=Windows&api=cl&D=AMD+Radeon+Pro+V320&testgroup=info)

#### Vega12 ( gfx904 ) {#vega12-gfx904}
| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 0x69A0 | 0x00 | | (Vega12 GL MXT) |
| 0x69A1 | 0x00 | | (Vega12 GL MXL) |
| 0x69A2 | 0x00 | | (Vega12 GL XL) |
| 0x69A3 | 0x00 | | (Vega12 Reserved) |
| 0x69AF &darr; | 0xC0 | Pro Vega 20 | (Vega12 XTA) |
| | 0xC1 | | (Vega12 MXT) |
| | 0xC3 | | (Vega12 MLP) |
| | 0xC7 | Pro Vega 16 | (Vega12 XLA) |
| | 0xCF | | (Vega12 XL) |
| | 0xFF | | (Vega12 LE) |

[Page Top](#page_index)

#### Vega20 ( gfx906 ) {#vega20-gfx906}
| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 0x66A0 | 00 | Instinct | |
| 0x66A1 &darr; | 00 | | (Server XT 32GB) |
| | 0x02 | MI50 32GB? | |
| | 0x03 | | |
| | 0x06 | Pro VII | (Vega20 WKS GL-XE) |
| 0x66A2 &darr; | 00 | | |
| | 0x02 | | |
| 0x66A3 | 00 | Pro Vega II (Duo) | |
| 0x66A4 | 00 | | |
| 0x66A7 | 00 | | |
| 0x66AF &darr; | C0 | | |
| | 0xC1 | VII | (Vega20 XT) |
| | 0xC3 | | |
| | 0xC7 | | |
| | 0xCF | | |
| | 0xF6 | | |
[Page Top](#page_index)

#### Arcturus ( gfx908 ) {#arcturus-gfx908}
| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 0x7388 | | | |
| 0x738C | | MI100[^5] | |
| 0x738E | | | |
| 0x7390 | | | (Arcturus VF) |

[Page Top](#page_index)

#### Aldebaran ( gfx90a ) {#aldebaran-gfx90a}
| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 0x7408 | | | |
| 0x740C | | | |
| 0x740F | | | |
| 0x7410 | | | (Aldebaran VF)[^aldebaran-vf] |

[Page Top](#page_index)

[^aldebaran-vf]: [[PATCH 1/1] drm/amdgpu: Add a new device ID for Aldebaran](https://lists.freedesktop.org/archives/amd-gfx/2021-April/062817.html)

### FAMILY_NV {#family_nv}
<!--
Navi LITE 13///\\\E9:00
-->

#### Navi10 ( gfx1010 ) {#navi10-gfx1010}
| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 0x66AF &darr; | 0x70 | | (Navi10 XTA, Fake DID?) |
| | 0x71 | | (Fake DID?) |
| | 0x72 | | (Fake DID?) |
| | 0x73 | | (Fake DID?) |
| | 0xF0 | | (Navi10 XTX, Fake DID?[^2]) |
| | 0xF1 | | (Navi10 XT, Fake DID?[^2]) |
| | 0xF2 | | (Navi10 XL, Fake DID?[^2]) |
| | 0xF3 | | (Navi10 XM, 12Gbps, Fake DID?) |
| | 0xF4 | | (12Gbps, Fake DID?) |
| | 0xF5 | | (12Gbps, Fake DID?) |
| | 0xF6 | | (Navi10 XME Fake DID?[^2]) |
| 0x7310 | 0x00 | Pro W5700X | (Navi10 GL-XTA) |
| 0x7312 | 0x00 | Pro W5700 | (Navi10 Pro-XL[^4]) |
| 0x7318 | 0x40 | | (Navi10 XTA) |
| 0x7319 | 0x40 | Pro 5700 XT | (Navi10 XTMA) |
| 0x731A | 0x40 | | |
| 0x731B | 0x40 | Pro 5700 | (Navi10 XGA) |
| 0x731E &darr; | 0xC6 | RX 5700XTB | for Blockchain |
| | 0xC7 | RX 5700B | for Blockchain |
| 0x731F &darr; | 0xC0 | RX 5700 XT 50th | (Navi10 XTX) |
| | 0xC1 | RX 5700 XT | (Navi10 XT) |
| | 0xC2 | RX 5600M | (Navi10 XME) |
| | 0xC3 | RX 5700M | |
| | 0xC4 | RX 5700 | (Navi10 XL) |
| | 0xC5 | RX 5700 XT | |
| | 0xC7 |  | |
| | 0xCA | RX 5600 XT | (Navi10 XLE) |
| | 0xCB | RX 5600 | (Navi10 XE) |
| | 0xE1 | | |
| | 0xE2 | | |
| | 0xE3 | | |
| | 0xE7 | | |
| | 0xEB | | |

[Page Top](#page_index)

[^2]: <https://cgit.freedesktop.org/~agd5f/linux/tree/drivers/gpu/drm/amd/powerplay/navi10_ppt.c?h=amd-staging-drm-next&id=fa34520c953b57a3ace3cfb3bfa3b3f1c8e0d414#n1698>
[^4]:[AMD Radeon Pro W5700 8 GB BIOS - TechPowerUp](https://www.techpowerup.com/vgabios/216287/216287)

#### Navi12 ( gfx1011 ) {#navi12-gfx1011}
| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 0x69B0 &darr; | 0x71 | | |
| | 0xF1 | | |
| | 0xF3 | | |
| 0x7360 &darr; | 0x40 | | |
| | 0x41 | Pro 5600M | |
| | 0xC1 | | |
| | 0xC3 | Pro V520 | |
| | 0xC7 | | (no video support)[^no-video-navi12] |
| 0x7362 | 0x71 | | (Navi12 VF) |
| | 0xC1 | | (Navi12 VF) |
| | 0xC3 | | (Navi12 VF) (AWS) |

[Page Top](#page_index)

[^pro5600m-youtube]: <https://www.youtube-nocookie.com/embed/unG5uGqefmA>
[^no-video-navi12]: [[PATCH] drm/amdgpu: disable VCN for Navi12 SKU](https://lists.freedesktop.org/archives/amd-gfx/2021-February/059663.html)

#### Navi14 ( gfx1012 ) {#navi14-gfx1012}
| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 0x67DF &darr; | 0x3C | | (Navi14 Pro-? Fake DID?) |
| | 0x3D | | (Navi14 Pro-? Fake DID?) |
| | 0x3E | | (Navi14 Pro-? Fake DID?) |
| | 0x3F | | (Navi14 Pro-XLM Fake DID?) |
| | 0xF2 | | (Navi14 XTM Fake DID?[^2]) |
| | 0xF3 | | (Navi14 XLM Fake DID?[^2]) |
| | 0xF4 | | (Navi14 XT Fake DID?[^2]) |
| | 0xF5 | | (Navi14 XL Fake DID??) |
| | 0xF6 | | (Navi14 XTX Fake DID?[^2]) |
| 0x7340 &darr; | 0x00 | Pro W5500X | Falcon GL-XTA |
| | 0x40 | Pro 5500M | (Navi14 ULA[^3]) |
| | 0x41 | Pro 5500 XT | Falcon XTA 8G |
| | 0x43 | Pro 5300M | |
| | 0x47 | Pro 5300 | Falcon XLXA | 
| | 0x70 | | |
| | 0xC1 | RX 5500M | (Navi14 XTM) |
| | 0xC3 | RX 5300M | (Navi14 XLM) |
| | 0xC5 | RX 5500 XT | (Navi14 XTX) |
| | 0xC7 | RX 5500 | (Navi14 XT) |
| | 0xC9 | RX 5500XTB | for Blockchain |
| | 0xCF | RX 5300 | (Navi14 XL?) |
| | 0xF2 | | |
| | 0xF3 | | |
| 0x7341 | 0x00 | Pro W5500 | (Navi14 Pro-XL) |
| 0x7343 | 0x00 | | |
| 0x7347 | 0x00 | Pro W5500M | (Navi14 Pro-XTM) |
| 0x734F | 0x00 | Pro W5300M | (Navi14 Pro-XLM) |

[Page Top](#page_index)

<!--
SubSystem ID? (Pro 5500M:0x020F, Pro 5300M:0x0210)
-->

[^3]:[Apple Pro 5500M 8 GB BIOS - TechPowerUp](https://www.techpowerup.com/vgabios/216534/apple-pro5500m-8192-191010)

#### Sienna Cichlid/Navi21 ( gfx1030 ) {#sienna_cichlid-gfx1030}

| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 0x731F &darr; | 0x50 | |
| | 0x51 | | |
| | 0xD0 | | |
| | 0xD1 | | |
| | 0xD3 | | |
| | 0xDF | | |
| 0x73A0 | 0x00 | |
| 0x73A1 | 0x00 | |
| 0x73A2 | 0x00 | |
| 0x73A3 | 0x00 | PRO W6800 |
| 0x73A4 | | |
| 0x73A5 | | |
| 0x73AB | 0x00 | |
| 0x73AC | | |
| 0x73AD | | |
| 0x73AE | | |
| 0x73AF | 0xC0 | RX 6900 XT |
| 0x73BD | | |
| 0x73BF &darr; | 0x40 | |
| | 0x41 | | |
| | 0xC0 | RX 6900 XT | |
| | 0xC1 | RX 6800 XT | |
| | 0xC3 | RX 6800 | |
| | 0xC7 | | |
| | 0xCF | | |
| | 0xD0 | | |
| | 0xD1 | | |
| | 0xD3 | | |


#### Navy Flounder/Navi22 ( gfx1031 ) {#navy_flounder-gfx1031}

| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 0x73C0 | 0x00 | |
| 0x73C1 | 0x00 | |
| 0x73C3 | 0x00 | |
| 0x73DF&darr; | 0x40 | |
| | 0x41 | |
| | 0xC0 | RX 6700 XT |
| | 0xC1 | RX 6700 XT |
| | 0xC2 | RX 6800M |
| | 0xC3 | RX 6800M |
| | 0xC5 | RX 6700 XT |
| | 0xCF | RX 6700M |
| | 0xDF | |
| 0x73DA | | |
| 0x73DB | | |
| 0x73DC | | |
| 0x73DD | | |
| 0x73DE | | |

[^14]: [P4 to Git Change 2042212 by kjayapra@1_HIPWS_LNX1_PAL on 2019/12/06 1… · ROCm-Developer-Tools/ROCclr@a35c1d2](https://github.com/ROCm-Developer-Tools/ROCclr/commit/a35c1d2f2d954363c7d2121d5334f9e7766beeae)
[^5]:<https://github.com/ROCm-Developer-Tools/aomp-extras/blob/f3d316fe64347e697a9789f0f2499fec50024db1/utils/bin/gputable.txt#L1867>

#### Dimgrey Cavefish/Navi23 ( gfx1032 ) {#dimgrey_cavefish-gfx1032}

| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 0x73E0 | 0x00 | |
| 0x73E1 | 0x00 | PRO W6600M[^amdgpu_ids-1337803] |
| 0x73E2 | | |
| 0x73E3 | 0x00 | PRO W6600[^amdgpu_ids-1337803] |
| 0x73E8 | | |
| 0x73E9 | | |
| 0x73EA | | |
| 0x73EB | | |
| 0x73EC | | |
| 0x73ED | | |
| 0x73EF | | |
| 0x73FF&darr; | | |
| | 0x40 | |
| | 0x41 | |
| | 0x42 | |
| | 0xC1 | RX 6600 XT |
| | 0xC3 | RX 6600M |
| | 0xC7 | |

### Beige Goby/Navi24 ( gfx1034 ) {#beige_goby-gfx1034}

| Device ID | Revision ID | Product Name | Memo |
| :--- | :--- | :---: | :---: |
| 0x7420 | | |
| 0x7421 | 0x00 | PRO W6500M[^amdgpu_ids-1337803] |
| 0x7422 | | |
| 0x7423 | 0x00 | PRO W6300M[^amdgpu_ids-1337803] |
| 0x743F &darr; | 0xC3 | RX 6500M[^amdgpu_ids-1337803] |
| | 0xCF | RX 6300M[^amdgpu_ids-1337803] | 

[^amdgpu_ids-1337803]: <http://repo.radeon.com/amdgpu/21.40.1/ubuntu/pool/main/libd/libdrm-amdgpu-common/libdrm-amdgpu-common_1.0.0.40501-1337803_all.deb>

{{< ref >}}

 * [DeviceInfoUtils.cpp - GPUOpen-Tools/common-src-DeviceInfo](https://github.com/GPUOpen-Tools/common-src-DeviceInfo/blob/master/DeviceInfoUtils.cpp)
 * [data/amdgpu.ids · master · Mesa / drm · GitLab](https://gitlab.freedesktop.org/mesa/drm/blob/master/data/amdgpu.ids)

{{< /ref >}} 
