---
title: "Nova Lake の GPU コアアーキテクチャは Xe3"
date: 2025-11-20T06:25:53+09:00
draft: false
categories: [ "Hardware", "Intel", "GPU" ]
tags: [ "Nova_Lake", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

インターネットの一部では Linux Kernel へのパッチを根拠に Intel の次世代プロセッサ *Nova Lake* に搭載される GPU を、*Panther Lake* や *Wildcat Lake* と同様の *Xe3* ではなく、データセンター向け GPU に採用される予定の *Xe3P* だとしているが、実際には *Xe3* だと思われる。  

 * [[PATCH 00/23] drm/xe: Add Xe3p support - Lucas De Marchi](https://lore.kernel.org/all/20251013-xe3p-v1-0-bfb74f038215@intel.com/)

ここでの新しい根拠は [intel/cm-compiler](https://github.com/intel/cm-compiler) の変更内容となる。  

 * [Add support for NVL-S and NVL-U devices · intel/cm-compiler@fc99a66](https://github.com/intel/cm-compiler/commit/fc99a661b7e65696fb530f779caa7eb96eccd7f6)
 * [Add support for xe3p device · intel/cm-compiler@5d3ad70](https://github.com/intel/cm-compiler/commit/5d3ad7006730c36926d96b49a8faeaf68b822e86)

Nova Lake のサポートを追加するパッチにおいて Nova Lake の GMD ID (Intel GPU のハードウェア IP バージョン) は、`nvl-ul/nvl-hx/nvl-s` が 30.4.4、`nvl-h/nvl-u` が 30.5.4、リリース ID は `xe3-lpg` とされている。  
その後に Xe3P、`cri (Crescent Island?)` のサポートを追加するパッチが公開されているが、GMD ID は 35.11.0 とされ、Nova Lake とはバージョンが大きく異なる。  
また、Xe3P では XMX (Xe Matrix eXtensions) で実行される行列積和演算命令 DPAS (Dot Product and Accumulate Systolic) に FP8/FP4 やマイクロスケーリングフォーマット (MXFP4/MXFP8 等) 関連の命令が追加されているが、それらの命令を Nova Lake はサポートしていない。  

 >         --- a/clang/lib/Driver/ToolChains/Arch/GenXPlatforms.cpp
 >         +++ b/clang/lib/Driver/ToolChains/Arch/GenXPlatforms.cpp
 >         @@ -15,6 +15,7 @@ using clang::driver::tools::GenX::encodeGmdId;
 >          
 >          // clang-format off
 >          static const std::unordered_map<std::string, uint32_t> ReleaseId = {
 >         +  {"xe3-lpg", encodeGmdId(30, 5, 4)},
 >            {"xe2-lpg", encodeGmdId(20, 4, 4)},
 >            {"xe2-hpg", encodeGmdId(20, 2, 0)},
 >            {"xe-lpgplus", encodeGmdId(12, 74, 4)},
 >         @@ -30,6 +31,11 @@ static const std::unordered_map<std::string, uint32_t> ReleaseId = {
 >          };
 >          
 >          static const std::unordered_map<std::string, uint32_t> DeviceId = {
 >         +  {"nvl-u", encodeGmdId(30, 5, 4)},
 >         +  {"nvl-h", encodeGmdId(30, 5, 4)},
 >         +  {"nvl-s", encodeGmdId(30, 4, 4)},
 >         +  {"nvl-hx", encodeGmdId(30, 4, 4)},
 >         +  {"nvl-ul", encodeGmdId(30, 4, 4)},
 >
 > {{< quote >}} [Add support for NVL-S and NVL-U devices · intel/cm-compiler@fc99a66](https://github.com/intel/cm-compiler/commit/fc99a661b7e65696fb530f779caa7eb96eccd7f6) {{< /quote >}}

Nova Lake の GPU 部において Xe3、Panther Lake や Wildcat Lake よりも進んだ部分となるのはあくまでもディスプレイエンジン、メディアエンジンだと思われる。  

