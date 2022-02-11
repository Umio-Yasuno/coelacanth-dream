---
title: "Cyan Skilfishシリーズ、Navi10_LITE、Navi12_LITE に Robin に Ariel"
date: 2022-02-05T06:58:12+09:00
draft: false
tags: [ "Cyan_Skilfish", "RDNA", "gfx1013" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
---

先日に *Cyan Skillfish/Skilfish* と *Navi10_LITE* を関連付けるパッチが投稿、公開されたが、さらなる情報を含んだパッチが新たに公開された。  
{{< link >}} [Cyan Skilfish と Navi10_LITE | Coelacanth's Dream](/posts/2022/01/24/cyan_skilfish_navi10_lite/) {{< /link >}}
どうでもいい話だが、*Cyan Skillfish/Skilfish* の名前が出てきた当初、*Skillfish* と *Skilfish* で表記が揺れており、*Skilfish (アブラボウズの英名)* が正しいと考え、自分はそちらに極力統一しているが、AMDGPUドライバーでは *Skillfish* がすっかり定着している。  
{{< link >}} [Linux Kernel に RDNA APU 「Cyan Skilfish」 をサポートするパッチが投稿される | Coelacanth's Dream](/posts/2021/07/21/amd-cyan_skilfish-rdna-apu/) {{< /link >}}

* [[PATCH 04/14] drm/amd/display: Basic support with device ID](https://lists.freedesktop.org/archives/amd-gfx/2022-February/074746.html)

## ROBIN+ {#robin}
パッチは前回のものをアップデートさせたような内容となっているが、追加される DeviceID が変更されており、前回は `0x1400` としていたのが、今回は `0x143F` としている。  
コメント部も興味を引く内容で、`0x143F` は `ROBIN+` の DeviceID であることを示している。  

 > 		-#define DEVICE_ID_NV_13FE 0x13FE  // CYAN_SKILLFISH
 > 		+#define DEVICE_ID_NV_NAVI10_LITE_P_13FE          0x13FE  // CYAN_SKILLFISH
 > 		+#define DEVICE_ID_NV_NAVI10_LITE_P_143F			0x143F // ROBIN+
 >
 > {{< quote >}} [[PATCH 04/13] drm/amd/display: Basic support with device ID](https://lists.freedesktop.org/archives/amd-gfx/2022-February/074734.html) {{< /quote >}}

コードネームの類と思われるが、自分は `ROBIN` の情報を全く持っていない。`ROBIN+` とあることから他のコードネームを持つ APU も同じ DeviceID に割り当てられている可能性もある。  
としても、今回のパッチで追加されたように、また `is_skillfish_series()` 関数における判定に使われていることからも *Cyan Skilfish* に関連することは確かと言える。  
`is_skillfish_series()` 関数を使用している部分からして、*ROBIN (0x143F)* もまたディスプレイコントローラーに DCN 2.01 を採用し、最大画面出力数は 2画面で変わらない。  
また、*Cyan Skilfish* はディスプレイコントローラー部において、eDP のバックライト調整機能 (Adaptive Backlight Modulation) や Panel Self Refresh 機能を担当する DMCU (Display Microcontroller Unit) を持たないが、同じ *RDNA/GFX10.1* 世代では *Navi10, Navi14* も持たないため、特別変わった点ではない。*RDNA/GFX10.1* 世代では *Navi12* だけが DMCU を持つ。  

 > 		+bool is_skillfish_series(struct amdgpu_device *adev)
 > 		+{
 > 		+	if (adev->asic_type == CHIP_CYAN_SKILLFISH || adev->pdev->revision == 0x143F) {
 > 		+		return true;
 > 		+	}
 > 		+	return false;
 > 		+}
 >
 > {{< quote >}} [[PATCH 04/13] drm/amd/display: Basic support with device ID](https://lists.freedesktop.org/archives/amd-gfx/2022-February/074734.html) {{< /quote >}}

*Navi10_LITE* には *Ariel* というコードネームも結びついている。*Ariel* は *Cyan Skilfish* 以前に内部で使われていたコードネームのようだが、現在では使われていない。  
どういった背景かは不明だが、ある APU を指すコードネームが多角的に増え、*Navi10_LITE, Navi12_LITE, Ariel, Cyan Skilfish, Robin* となったのだろう。  

 > 			SWDEV-132899 - [OCL][GFX10] passing "force-wgp-mode" option to Finalizer to enable WGP mode by default on gfx10+
 > 			and allow GPU_ENABLE_WGP_MODE to control the WGP/CU mode for HSAIL/SC path as well.
 > 			- also for Ariel (Navi10Lite) the wave32 should be disabled in LC but allow GPU_ENABLE_WAVE32_MODE control it for testing if needed.
 >
 > {{< quote >}} [P4 to Git Change 1757990 by asalmanp@asalmanp-ocl-stg on 2019/03/18 2… · ROCm-Developer-Tools/ROCclr@4e892db](https://github.com/ROCm-Developer-Tools/ROCclr/commit/4e892db7dd122d99fac6408049bcec03b71340a2) {{< /quote >}}

*Cyan Skilfish* は *gfx1013* として LLVM などにはサポートが追加されているが、Mesa3D の *RadeonSI (OpenGL)* 、*RADV (Vulkan)* には未だサポートが追加されていない。  
{{< link >}} [HWレイトレーシングをサポートする RDNA APU 「Cyan Skilfish (gfx1013)」 | Coelacanth's Dream](/posts/2021/08/01/cyan_skilfish-apu-gfx1013/) {{< /link >}}
どういった製品、SKU で登場するのか、そして普通の APU として使用可能どうかも、複数のコードネームと合わせてまだまだ不明。  

最後に、`ROBIN+` のコメント部は他のパッチで消されている。*Navi10_LITE* の名も、AMDGPUドライバーに一度現れて消え、そしてまた現れている。  
あまり姿を見せられないカスタム APU なのだろうが、それでも AMDGPUドライバーではサポート作業を進めている。それだけに不明な部分が増えているようにも思う。  

* [[PATCH 14/14] SWDEV-321758 - dc: Code clean](https://lists.freedesktop.org/archives/amd-gfx/2022-February/074757.html)

{{< ref >}}
* [[PATCH 3/3] drm/amd: Add DM DMCU support](https://lists.freedesktop.org/archives/amd-gfx/2018-September/026550.html)
{{< /ref >}}
