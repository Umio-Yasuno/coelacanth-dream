---
title: "もう 1つの VanGogh APU? DeviceID: 0x1435"
date: 2022-09-15T08:50:11+09:00
draft: false
categories: [ "Hardware", "AMD", "APU" ]
tags: [ "VanGogh", "Linux_Kernel", "Cyan_Skilfish" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD の Wayne Lin 氏により 2022/09/14 付で公開された AMDGPU Display Core (Core) ドライバコンポーネントへのパッチセットに、もう 1つの *VanGogh APU* らしき APU に向けたパッチが含まれていた。  
パッチは AMD の Pavle Kotarac 氏により作成されている。  
パッチには DeviceID の情報が記述されているが、現時点では他で確認されていない AMD APU の DeviceID であり、ほとんど正体不明な APU となる。  

 * [[PATCH V3 25/47] drm/amd/display: Added new DCN301 Asic Id](https://lists.freedesktop.org/archives/amd-gfx/2022-September/084326.html)
 * [[PATCH V3 26/47] drm/amd/display: Removing 2 phys](https://lists.freedesktop.org/archives/amd-gfx/2022-September/084325.html)

## VanGogh APU {#vgh}
*VanGogh APU* は *Zen 2 CPU + RDNA 2 GPU* の構成を採る AMD APU であり、**Steam Deck** に CPU名 **Custom APU 0405**、GPU名 **Custom GPU 0405** として搭載されている。  
メディアエンジンには VCN 3.0、ディスプレイエンジンには DCN 3.0.1 が採用されている。  
現在 *VanGogh APU* としてサポートされているのは、DeviceID では `0x163F` のものだけとなっている。  

## DeviceID: 0x1435 {#1435h}
今回のパッチで、もう 1つの DCN 3.0.1 を採用する *VanGogh APU* として DeviceID: `0x1435` が追加された。  
パッチの対象ファイルは AMDGPU DC の `drivers/gpu/drm/amd/display/include/dal_asic_id.h` ヘッダと DCN 3.0.1 のリソースが記述された `drivers/gpu/drm/amd/display/dc/dcn301/dcn301_resource.c` となっている。  
パッチの差分に含まれている、DeviceID の値的には近い `0x143F` は *Robin* というコードネームも確認されている *Cyan Skilfish/Skillfish (Navi10_Lite, Navi12_Lite, GC 10.1.3, GC 10.1.4, gfx1013)* に割り当てられた DeviceID。  

 > 		 #define DEVICE_ID_NV_143F 0x143F
 > 		 #define FAMILY_VGH 144
 > 		 #define DEVICE_ID_VGH_163F 0x163F
 > 		+#define DEVICE_ID_VGH_1435 0x1435
 > 		 #define VANGOGH_A0 0x01
 > 		 #define VANGOGH_UNKNOWN 0xFF
 > 		
 >
 > {{< quote >}} [[PATCH V3 25/47] drm/amd/display: Added new DCN301 Asic Id](https://lists.freedesktop.org/archives/amd-gfx/2022-September/084326.html) {{< /quote >}}

AMDGPUドライバの他の部分に DeviceID: `0x1435` に関連するコードは追加されていないが、AMDGPUドライバはファームウェア等から読み取れる IPバージョンベースのサポートに移行しているため、最小限の変更となっている。  
変更が小さい理由には、*VanGogh APU* (DeviceID: 0x163F) と機能的にはほとんど同じだから、ということも考えられる。  

DeviceID: `0x163F` と `0x1435` の違いには、ディスプレイエンジン部の PLL (Phase-Locked Loop) PHY の数だけが明らかとなっている。  
DeviceID: `0x163F` では PLL 4基となっているが、`0x1435` では 2基となる。PLL 2基という小規模の構成は、比較的最近の AMD APU では *Cyan Skilfish/Skillfish* のディスプレイエンジン DCN 2.0.1 にしか見られない。  

 > 		[WHY]
 > 		New dcn301 has 2 less phys
 >      [...]
 > 		+	if (dc->ctx->asic_id.chip_id == DEVICE_ID_VGH_1435)
 > 		+		res_cap_dcn301.num_pll = 2;
 > 		
 > {{< quote >}} [[PATCH V3 26/47] drm/amd/display: Removing 2 phys](https://lists.freedesktop.org/archives/amd-gfx/2022-September/084325.html) {{< /quote >}}

DeviceID: `0x1435` が `0x163F` と同じダイで一部を無効化した APU である可能性と、各種 IP は `0x163F` と同じだが一部規模を削減した新規ダイである可能性が考えられるが、現状答えは出ていない。  
単純に `num_pll` でソースコードを検索したところ、`num_pll` の値を参照してるコードが見つからなかったため、パッチによる影響があるかは疑問である。  
それでも実際のハードウェアの情報を反映した結果であることに変わりはないと思われる。  

*VanGogh APU* に近い APU には、Samsung Exynos 2200 SoC に採用されている *VanGogh_Lite (Mariner, Gopher)* があるが、公開されている Linux Kernel のソースコードから AMD IP ではないディスプレイエンジンを採用している可能性が高い。  
Magic Leap 2 に搭載されている *Zen 2 CPU 4-Core + RDNA 2 GPU* という構成のカスタム APU/SoC も *VanGogh APU* と近いと言えるが、ドライバのソースコード等がまだ公開されていないため、これも確証はない。  

あれこれ書いても、結局の所 DeviceID: `0x1435` の *VanGogh APU* は、パッチがこのタイミングで投稿されたことも含めて、ほとんど謎の APU ということに尽きる。  

{{< ref >}}
 * [Cyan Skilfishシリーズ、Navi10_LITE、Navi12_LITE に Robin に Ariel | Coelacanth's Dream](/posts/2022/02/05/cyan_skilfish-robin-apu/)
 * [VanGogh_LITE (Mariner, Gopher) 向け AMDGPU ドライバーのソースコードを読んでみる | Coelacanth's Dream](/posts/2022/06/14/vangogh_lite/)
 * [Enterprise augmented reality (AR) platform designed for business | Magic Leap](https://www.magicleap.com/)
{{< /ref >}}
