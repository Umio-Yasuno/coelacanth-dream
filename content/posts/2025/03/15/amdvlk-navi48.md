---
title: "AMDVLK ドライバーが Navi48 をサポート"
date: 2025-03-15T20:08:28+09:00
draft: true
categories: [ "Hardware", "AMD", "GPU" ]
tags: [ "GPUOpen", "GFX12", "RDNA_4" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD がオープンソースで開発している AMDVLK ドライバーが *Navi48* に対応した。*Navi48* は RX 9700 /XT のベースとなる GPU のコードネームとされている。  

 * [Update llpc from commit 5f1b27d9 · GPUOpen-Drivers/llpc@188bbf6](https://github.com/GPUOpen-Drivers/llpc/commit/188bbf6a5b9403813e51d39f6fc8429550dbf267)
 * [Update gpurt from commit 55ac0f61 · GPUOpen-Drivers/gpurt@f734985](https://github.com/GPUOpen-Drivers/gpurt/commit/f734985ebc31f471c376ed0cb217f43bdd40ee17)

## v_dot2_f16_f16/v_dot2_bf16_bf16 の精度 {#dot}
ドキュメントには特に書かれていないが、*RDNA 4* では `V_DOT2_F16_F16/V_DOT2_BF16_BF16` 命令の精度が改善されたらしく、*RDNA 3* では満たせなかった Vulkan API 要求する精度を満たすことができるとしている。  
*RDNA 4* では行列演算専用ユニットが CU 内に新設されたが、ドット積命令もそこで実行するようになったのかもしれない。  

 >         +    // Use a chain of v_dot2_f16_f16/v_dot2_bf16_bf16 on gfx12+.
 >         +    //
 >         +    // Note: GFX11 has this instruction, but its precision doesn't satisfy Vulkan requirements.
 >         +    //
 >         +    // Note: GFX10 chips may have v_dot2_f32_f16, which we could consider generating in cases where bitexact results
 >         +    //       are not required.
 >
 > {{< quote >}} [Update llpc from commit 5f1b27d9 · GPUOpen-Drivers/llpc@188bbf6](https://github.com/GPUOpen-Drivers/llpc/commit/188bbf6a5b9403813e51d39f6fc8429550dbf267) {{< /quote >}}

## RT IP 3.0, 3.1 {#rt_ip}
レイトレーシングユニットとして RT IP 3.0 と 3.1 のサポートが今回追加されている。  

 >         +#if GPURT_BUILD_RTIP3
 >         +    RtIp3_0         = 0x4,  ///< Added high precision box node, HW instance node, dual intersect ray, BVH8 intersect
 >         +                            ///  ray, LDS stack push 8 pop 1, and LDS stack push 8 pop 2
 >         +#endif
 >              RtIpReserved    = 0x5,  ///< Special value, should not be used
 >         +#if GPURT_BUILD_RTIP3_1
 >         +    RtIp3_1         = 0x6,  ///< Added improved bvh footprints (change to node pointer, 128 Byte primitive structure
 >         +                            ///  format, 128 Byte Quantized box node, obb support, wide sort)
 >         +#endif
 >
 > {{< quote >}} [Update gpurt from commit 55ac0f61 · GPUOpen-Drivers/gpurt@f734985](https://github.com/GPUOpen-Drivers/gpurt/commit/f734985ebc31f471c376ed0cb217f43bdd40ee17) {{< /quote >}}

RT IP 3.0 では高精度 (64-bit) なボックスノードのフォーマットへの対応、Dual BVH4 Intersect Ray、BVH8 Intersect Ray、新たなスタック管理命令に対応し、  
RT IP 3.1 ではそこからノードポインターの変更、量子化されたボックスノード、OBB (Oriented Bounding Box)、ワイドソートに対応している。  
*RDNA 4 アーキテクチャ* に関する発表内容と `pal` のコード[^rtip3_1] から、*Navi48* は RT IP 3.1 を採用している。  
RT IP 3.0 に関して明言できる資料等は無いが、内容からして PS5 Pro で採用されているレイトレーシングユニットを指すのではないかと思われる。  

[^rtip3_1]: <https://github.com/GPUOpen-Drivers/pal/blob/04bc1e796dd15fc90fff8fa826d32e431d8722f6/src/core/hw/gfxip/gfx12/gfx12Device.cpp#L351>
