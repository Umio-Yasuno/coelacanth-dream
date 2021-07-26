---
title: "RADVドライバーに NGGカリングが実装"
date: 2021-07-26T22:33:40+09:00
draft: false
tags: [ "RADV", "NGG" ]
# keywords: [ "", ]
categories: [ "Software", "AMD", "GPU" ]
noindex: false
# summary: ""
---

オープンソースで開発される AMD GPU 向け Vulkanドライバー **RADV** に、先日 NGGカリングを実装するパッチがマージされた。  

 * [radv: Implement NGG culling (!10525) · Merge requests · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/10525)

当マージリクエストを作成した、**RADV** の開発者である [Timur Kristóf](https://gitlab.freedesktop.org/Venemo) 氏は *NGG* と *NGGカリング* について解説を行っており、理解を深めるため個人的に整理し、書き起こしてみる。また、以前書いた内容と被る部分がある。  
{{< link >}} [ACOバックエンドでも NGG をサポートする動き | Coelacanth's Dream](/posts/2020/10/04/aco-ngg-gfx10/) {{< /link >}}

## NGG {#ngg}

*NGG* は *GFX10/RDNA* 世代の GPU から *サポート* されているハードウェアにおけるシェーダーステージであり、ピクセルシェーダー (Pixel Shader, PS) の入力に用いられるオンチップバッファ、パラメーターキャッシュ (Parameter cache, PC) が直接接続されている。  
*NGG* ステージでは、オブジェクトの頂点処理を行なうバーテックスシェーダー (Vertex Shader, VS)、頂点の増減を行なうジオメトリシェーダー (Geometry Shader, GS) が実行でき、将来的にはメッシュシェーダー (Mesh Shader, MS) も *NGG* ステージで実行するとしている。  
従来の AMD GPU のハードウェアシェーダーパイプラインでは、GS の実行結果を直接 PS に渡すことができず、一度 VRAM に結果を書き込み、そこから PS が受け取るバッファに書き込む必要があったが、*NGG* はその手間を解消するものとなる。  

*NGG* ステージで実行されるシェーダーは頂点だけでなくトライアングルなどのプリミティブも変更することができ、これを活用したのが *NGGカリング* とされる。  

*NGG* は *GFX9/Vega* 世代からハードウェアに実装されたが、具体的な理由は明かされていないものの、最終的に *GFX9/Vega* 世代のサポートは諦め、*GFX10/RDNA* からのサポートに注力されることとなった。[^ngg-gfx9]  

[^ngg-gfx9]: [Making a GDS Allocation for NGG](https://lists.freedesktop.org/archives/amd-gfx/2018-August/025320.html)

### Parameter cache {#pc}

*NGGカリング* の前にパラメーターキャッシュについて触れると、  
パラメーターキャッシュの規模は APU/GPU ごとに異なり、 [src/amd/common/ac_gpu_info.c](https://gitlab.freedesktop.org/mesa/mesa/-/blob/9da4590df8b7d08d51464874987313d230adfee8/src/amd/common/ac_gpu_info.c#L980-1007) にパラメーターキャッシュラインの数が記述されている。  
*GFX9/Vega* 世代の dGPU、*Vega10・Vega12・Vega20* は最も多い 2048、同世代の *Raven・Raven2・Renoir* APU では 1024。  
*GFX10/RDNA* 世代では、*Navi10・Navi12* は *GFX9* 世代の APU と同じ 1024、*Navi14* は半分の 512、*GFX10.3/RDNA 2* 世代からは GPUコア部の規模がだいぶ異なる *Sienna Cichlid (Navi21)・Navy Flounder (Navi22)・Dimgrey Cavefish (Navi23)* で同じライン数を搭載、*Beige Goby* のみは *Navi14* 同じ 512 という数を取り、  
*GFX10.3/RDNA 2* 世代の APU である *VanGogh* と *Yellow Carp* は最も少ない 256 となっている。  

 > 		   if (info->chip_class >= GFX9 && info->has_graphics) {
 > 		      unsigned pc_lines = 0;
 > 		
 > 		      switch (info->family) {
 > 		      case CHIP_VEGA10:
 > 		      case CHIP_VEGA12:
 > 		      case CHIP_VEGA20:
 > 		         pc_lines = 2048;
 > 		         break;
 > 		      case CHIP_RAVEN:
 > 		      case CHIP_RAVEN2:
 > 		      case CHIP_RENOIR:
 > 		      case CHIP_NAVI10:
 > 		      case CHIP_NAVI12:
 > 		      case CHIP_SIENNA_CICHLID:
 > 		      case CHIP_NAVY_FLOUNDER:
 > 		      case CHIP_DIMGREY_CAVEFISH:
 > 		         pc_lines = 1024;
 > 		         break;
 > 		      case CHIP_NAVI14:
 > 		      case CHIP_BEIGE_GOBY:
 > 		         pc_lines = 512;
 > 		         break;
 > 		      case CHIP_VANGOGH:
 > 		      case CHIP_YELLOW_CARP:
 > 		         pc_lines = 256;
 > 		         break;
 > 		      default:
 > 		         assert(0);
 > 		      }
 >
 > {{< quote >}} [src/amd/common/ac_gpu_info.c · 29f264f25804eeea962f21c29c39050c3fc1663d · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/blob/29f264f25804eeea962f21c29c39050c3fc1663d/src/amd/common/ac_gpu_info.c#L1013-1042) {{< /quote >}}

パラメーターキャッシュラインの数から、GPUコア部やメモリインターフェイスの規模と一緒にスケールするものではなく、ある程度調整することができ、ターゲットとする性能やシミュレーション結果、製造コスト等から決定されるのではないかと考えられる。  
NGG を用いたパイプラインを実行する際にパラメーターキャッシュの一部を割り当てる形となり、[AMDVLK GPUOpen](/tags/gpuopen)ドライバーでは *GFX10/RDNA* 世代の場合、バッチ 1つがパラメータキャッシュ全体の 1/3 を使用することを推奨値としている。[^pc-amdvlk]  

[^pc-amdvlk]: <https://github.com/GPUOpen-Drivers/pal/blob/ad699adac6f9f331bbc454050f6b40d1549ce752/src/core/hw/gfxip/gfx9/gfx9SettingsLoader.cpp#L123-L159>

パラメータキャッシュの規模が NGG の実効性能に影響すると思われるが、どれ程影響するかは不明で、パッチ内で触れられたこともない。  
恐らく、パラメーターキャッシュの規模を決めているのは GPU のハードウェア開発チームであり、ドライバーを開発するソフトウェアチームは決定までの具体的な情報を知りにくいのだと思われる。  

## NGG culling {#nggc}

最終的には表示されない、不要なトライアングルを削除する処理が *カリング* となる。  
カリング処理は従来ラスタライザで実行されていたが、ラスタライザは固定機能部であり、大量のプリミティブが作成された際、ラスタライザがボトルネックとなることがあった。  
小さいトライアングルが大量に存在すると、GCN/RDNA GPU ではラスタライズ処理の効率が特に低下していた。  

だが *NGG* ステージを用いれば、固定機能部であるラスタライザではなくシェーダーでプリミティブをスキャンすることができ、同時に不要なプリミティブを削除することが可能となる。  

今回のマージリクエストではカリングアルゴリズムとして、カメラに映らない面のトライアングルを削除する *Front/back face culling* 、ビューポート (画面) の外側にあるトライアングルを削除する *Frustrum culling* 、どのピクセルにも映らないような小さいトライアングルを削除する *Small primitive culling* が実装されている。  

*NGG/NGGカリング* は *GFX10/RDNA* 、*GFX10.3/RDNA 2* 世代の GPU でサポートされているが、RB (RenderBackend) を 1基しか持たないような規模の小さい APU/GPU では有効されず、また *Navi14* ではハードウェアに存在するバグによって NGG を有効化することができない。  
前者の条件に一致する APU/GPU はまだ明らかでないが、そのような *RDNA/RDNA 2 アーキテクチャ* を採用した非常に小規模な APU/GPU が計画されているのかもしれない。  

 > 		   device->use_ngg = device->rad_info.chip_class >= GFX10 &&
 > 		                     device->rad_info.family != CHIP_NAVI14 &&
 > 		                     !(device->instance->debug_flags & RADV_DEBUG_NO_NGG);
 > 		
 > {{< quote >}} [src/amd/vulkan/radv_device.c · cadf2d63b736b610728d508ed507551dd74ba16a · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/-/blob/cadf2d63b736b610728d508ed507551dd74ba16a/src/amd/vulkan/radv_device.c#L677-679) {{< /quote >}}

