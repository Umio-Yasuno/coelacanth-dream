---
title: "Intel Gen12 GPU は AV1コーディックのHWデコードをサポート"
date: 2020-07-09T16:49:24+09:00
draft: false
tags: [ "Gen12", "Rocket_Lake", "DG1", "Tiger_Lake" ]
keywords: [ "", ]
categories: [ "Hardware", "Intel", "GPU" ]
noindex: false
---

既に Youtube 等では採用されているオープンであり、ロイヤリティーフリーでもある動画コーディック、  
**AV1 (AOMedia Video 1)** のハードウェアデコードをサポートすることが、[intel/media-driver](https://github.com/intel/media-driver)に追加されたパッチからわかった。  
{{< link >}}[[Decode] This enables HW AV1 decode acceleration on Gen12 · intel/media-driver@9491998](https://github.com/intel/media-driver/commit/9491998f40d496fc458d282f213c0e9e945b8062){{< /link >}}

コードを見るに、AV1 8-bit 4:2:0、AV1 10-bit 4:2:0 に対応するものと思われる。[^gen12-av1-1]  
解像度に関しては、他の HEVC、VP9コーディック同様に 16K に対応するようにも読めるが、実際の仕様に反映されるかは不明。[^gen12-av1-2]  

<!-- (Y'CbCr) -->

[^gen12-av1-1]: [media-driver/media_sku_wa_g12.cpp at 9491998f40d496fc458d282f213c0e9e945b8062 · intel/media-driver](https://github.com/intel/media-driver/blob/9491998f40d496fc458d282f213c0e9e945b8062/media_driver/linux/gen12/ddi/media_sku_wa_g12.cpp#L99)
[^gen12-av1-2]: [media-driver/media_libva_caps_g12.h at 9491998f40d496fc458d282f213c0e9e945b8062 · intel/media-driver](https://github.com/intel/media-driver/blob/9491998f40d496fc458d282f213c0e9e945b8062/media_driver/linux/gen12/ddi/media_libva_caps_g12.h#L174)

AV1コーディックのHWデコードに対応するプロセッサは、現時点では *Mediatek Dimensity 1000 5G SoC*  [^mediatek-av1] が発表されているのみで、一部では既に採用こそされているがデコードは負荷の掛かるソフトウェアデコードで行なわれているため、本格的な普及にはまだ時間が掛かると見られていた。  
しかし、Intel の次世代プロセッサ *Tiger Lake* 、*Rocket Lake* が、負荷が軽く、省電力、高速で行なえるHWデコードに対応することで、AV1コーディックの普及を後押しするものと思われる。  
GPUベンダーである、NVIDIA、AMD がどの世代で対応するかはまだ不明であり、メディア機能においては Intel GPU がリードを取ったと見れる。  

[^mediatek-av1]: [MediaTek to Enable Cutting-edge AV1 Video Codec Technology on Android Smartphones | MediaTek](https://www.mediatek.com/news-events/press-releases/mediatek-to-enable-cutting-edge-av1-video-codec-technology-on-android-smartphones)
