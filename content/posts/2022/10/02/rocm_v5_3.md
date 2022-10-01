---
title: "AMD、ROCm v5.3 をリリース"
date: 2022-10-02T06:52:29+09:00
draft: false
categories: [ "Software", "AMD", "GPU" ]
tags: [ "ROCm", "GFX11" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD より 2022/09/30 付で ROCm v5.3 がリリースされた。  

今回のリリースではサポート対象の OS、Linux ディストリから Ubuntu 18.04.x が外され、Ubuntu 22.04.x が追加された。  
また各種ライブラリとヘッダファイルのファイル配置が変更された。ラッパーとなるヘッダファイルとシンボリックリンクが同時に追加されたため、ソースコードを変更しなくとも警告メッセージが出るだけとなる。  
この後方互換性は次回のメジャーバージョンアップデート (ROCm v6.0?) まで維持される予定。  

 * [About This Document](https://docs.amd.com/bundle/ROCm-Release-Notes-v5.3/page/About_This_Document.html)
 * [What’s New in This Release](https://docs.amd.com/bundle/ROCm-Release-Notes-v5.3/page/What%E2%80%99s_New_in_This_Release.html)
 * [Deprecations and Warnings](https://docs.amd.com/bundle/ROCm-Release-Notes-v5.3/page/Deprecations_and_Warnings.html)

また、HPC 向けに *Introduction to AMD Instinct MI250 High Performance Computing and Tuning Guide* が公開された。  

 * [Introduction to AMD Instinct MI250 High Performance Computing and Tuning Guide](https://docs.amd.com/bundle/AMD-Instinct-MI250-High-Performance-Computing-and-Tuning-Guide-v5.3/page/Introduction_to_AMD_Instinct_MI250_High_Performance_Computing_and_Tuning_Guide.html)

ROCmプラットフォームとしてサポートする AMD GPU に変更はなく、*RDNA 2/GFX10.3* GPU は公式的には **Radeon Pro W6800 (Navi21 GL-XL)** と **Radeon Pro V620 (Navi21 GL-XE)** のみとなっている。  
しかし各種ライブラリでは *RDNA 3/GFX11* へのサポートが進み始めており、*RDNA 2/GFX10.3* では *Navi21/Sienna Cichlid/gfx1030* のみのサポートに留まっていたライブラリでも複数の *RDNA 3/GFX11* の GPU ID がサポートターゲットに追加されている。  

 * [Enabling gfx1100 and gfx1101 support (#291) · ROCmSoftwarePlatform/rocRAND@a5b942d](https://github.com/ROCmSoftwarePlatform/rocRAND/commit/a5b942d2d8cab62bc286ffb8953b8db9f9d650a1)
 * [[OpenMP][AMDGPU] Enable OpenMP device runtime build for gfx110[0123] · llvm/clangir@db021ab](https://github.com/llvm/clangir/commit/db021abf33d325303500c883902be9150bcd3c22)
 * [Enabling gfx1100 and gfx1101 support (#291) · ROCmSoftwarePlatform/rocRAND@a5b942d](https://github.com/ROCmSoftwarePlatform/rocRAND/commit/a5b942d2d8cab62bc286ffb8953b8db9f9d650a1)
 * [Add gfx1100 and gfx1102 to default AMDGPU_TARGETS · ROCmSoftwarePlatform/rocFFT@11561f8](https://github.com/ROCmSoftwarePlatform/rocFFT/commit/11561f8d7e2c2de620159013bff96bf7b72aa972)
 * [Merge pull request #1521 from TonyYHsieh/navi31_v1 · ROCmSoftwarePlatform/Tensile@b55e049](https://github.com/ROCmSoftwarePlatform/Tensile/commit/b55e049a220321d57d1c2c97ce5db39138211d31)
 * [Add initial compatibility for gfx11xx · ROCmSoftwarePlatform/rocWMMA@e8fb0d5](https://github.com/ROCmSoftwarePlatform/rocWMMA/commit/e8fb0d579174531d4e9c99158a715a79f3480a3d)

リリースノートでは触れられていないものの、機械学習向けライブラリ [MIOpen](https://github.com/ROCmSoftwarePlatform/MIOpen) ではバックエンドに HIP と OpenCL の両方をサポートしていたが、今回のリリースに含まれるコミットで OpenCL バックエンドを非推奨とするメッセージを出力するようになった。  
[RadeonImageFilter](https://github.com/GPUOpen-LibrariesAndSDKs/RadeonImageFilter) や [RadeonML](https://github.com/GPUOpen-LibrariesAndSDKs/RadeonML) では MIOpen OpenCL バックエンドが用いられていた。  

 * [MIOpen add OpenCL backend deprecation Message (#1664) · ROCmSoftwarePlatform/MIOpen@b6f49fa](https://github.com/ROCmSoftwarePlatform/MIOpen/commit/b6f49fa3a18a30f0efe6336378b099c699816d47)

RadeonImageFilter も RadeonML も今は更新が止まっており、[HIPRTSDK](https://github.com/GPUOpen-LibrariesAndSDKs/HIPRTSDK) のリリースや Blender Cycles レンダリングエンジンでの HIP API サポートから見るに、今後は Windows/Linux 両方のサポートを想定したライブラリでも HIP API の採用がメインになるのではないかと思われる。  

{{< ref >}}
 * [Blender 3.0 and AMD: Creating a New Performance Pa... - AMD Community](https://community.amd.com/t5/radeon-pro-graphics/blender-3-0-and-amd-creating-a-new-performance-path/ba-p/499333)
{{< /ref >}}
