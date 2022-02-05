---
title: "Cyan Skilfish と Navi10_LITE"
date: 2022-01-24T06:56:23+09:00
draft: false
tags: [ "Cyan_Skilfish", "gfx1013", "RDNA" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
---

AMDGPU Display Core (DC) ドライバの新しいパッチが投稿、公開された。  
そしてその中に *Cyan Skilfish/Skillfish APU* の DeviceID を追加するパッチが含まれており、同時に *NAVI10_LITE* のコードネームも出されている。  

 * [[PATCH 13/24] drm/amd/display: Basic support with device ID](https://lists.freedesktop.org/archives/amd-gfx/2022-January/074104.html)

## NAVI10_LITE {#navi10_lite}
*Cyan Skilfish* のサポートを AMDGPUドライバに追加するパッチは 2021/07 に投稿され、その時追加された DeviceID は `0x13FE` のみだった。  
{{< link >}} [Linux Kernel に RDNA APU 「Cyan Skilfish」 をサポートするパッチが投稿される | Coelacanth's Dream](/posts/2021/07/21/amd-cyan_skilfish-rdna-apu/#v2) {{< /link >}}
今回のパッチで `0x1400` が *Cyan Skilfish* の DeviceID として追加されたが、同時に *NAVI10_LITE (NV_NAVI10_LITE_P)* に割り当てられた DeviceID ともしている。  

 > 		+#define DEVICE_ID_NV_NAVI10_LITE_P_1400			0x1400 // CYAN_SKILLFISH
 > 		+
 >
 > {{< quote >}} [[PATCH 13/24] drm/amd/display: Basic support with device ID](https://lists.freedesktop.org/archives/amd-gfx/2022-January/074104.html) {{< /quote >}}

しかし *Cyan Skilfish* と異なるコードネームに関しては、*NAVI12_LITE (NV_NAVI12_LITE_P_A0)* と `external_rev_id/eChipRev` が一致する。`external_rev_id/eChipRev` は AMD APU/GPU ASIC に割り当てられた ID であり、  
{{< link >}} [Cyan Skilfish APU のディスプレイエンジンをサポートするパッチが投稿される　―― NAVI12_LITE | Coelacanth's Dream](/posts/2021/09/28/cyan_skilfish-display-support/#navi12_lite) {{< /link >}}

*Cyan Skilfish* には *NAVI10_LITE* と *NAVI12_LITE*、2つのコードネームも結び付けられていることとなるが、*Cyan Skilfish* 自体も 2種類のバージョンが存在する。  

### Cyan Skilfish/2 {#cyan_skilfish_v2}

AMDGPUドライバのコードから、*Cyan Skilfish* と *Cyan Skilfish2* が存在するとされ、読み込むファームウェアもそれぞれで別に用意される。  
{{< link >}} [Linux Kernel に RDNA APU 「Cyan Skilfish」 をサポートするパッチが投稿される | Coelacanth's Dream](/posts/2021/07/21/amd-cyan_skilfish-rdna-apu/#v2) {{< /link >}}
上で DeviceID: `0x13FE` が *Cyan Skilfish* に割り当てられていると書いたが、厳密には *Cyan Skilfish2* に割り当てられている DeviceID となる。  
今回のパッチが投稿されるまで *Cyan Skilfish* の DeviceID は `0x13FE` のみだったため、AMDGPUドライバは *Cyan Skilfish2* をターゲットにサポートを追加していたと思われる。`external_rev_id/eChipRev` も *Cyan Skilfish2* のものだろう。  

 > 		+	case CHIP_CYAN_SKILLFISH:
 > 		+		if (adev->pdev->device == 0x13FE)
 > 		+			adev->apu_flags |= AMD_APU_IS_CYAN_SKILLFISH2;
 > 		+		break;
 >
 > {{< quote >}} [[PATCH 03/29] drm/amdgpu: add cyan_skillfish asic type](https://lists.freedesktop.org/archives/amd-gfx/2021-July/066805.html) {{< /quote >}}

要は *Cyan Skilfish* に *NAVI10_LITE*、*Cyan Skilfish2* に *NAVI12_LITE* が結び付けられると考えられる。  
*Cyan Skilfish/2* の違いには PSP (Platform Security Processor) 部があり、AMDドライバ中では PSP部 (v11.0.8) において *Cyan Skilfish2* のみをサポートしている。おそらく *Cyan Skilfish* では PSP部に何かしらの問題 (バグ等) があり、それが *Cyan Skilfish2* で修正されたのではないかと思われる。  
`external_rev_id/eChipRev` では、*NAVI10_LITE_P_A0 /NAVI10_LITE_P_B0, /NAVI12_LITE_P_A0* は連番になっており、*Cyan Skilfish2* にリビジョンが進んだ *NAVI12_LITE_P_A0* が割り当てられるのは自然なものに感じられる。  

 > 		+#define FAMILY_NV 143 /* DCN 2*/
 > 		+
 > 		+enum {
 > 		+	NV_NAVI10_P_A0      = 1,
 > 		+	NV_NAVI12_P_A0      = 10,
 > 		+	NV_NAVI14_M_A0      = 20,
 > 		+	NV_NAVI21_P_A0      = 40,
 > 		+	NV_NAVI10_LITE_P_A0 = 0x80,
 > 		+	NV_NAVI10_LITE_P_B0 = 0x81,
 > 		+	NV_NAVI12_LITE_P_A0 = 0x82,
 > 		+	NV_NAVI21_LITE_P_A0 = 0x90,
 > 		+	NV_UNKNOWN          = 0xFF
 > 		+};
 > 		+
 >
 > {{< quote >}} [[PATCH 316/459] drm/amd/display: Add DCN2 and NV ASIC ID](https://lists.freedesktop.org/archives/amd-gfx/2019-June/035497.html) {{< /quote >}}

とはいえ、コードネームを *Cyan Skilfish* に変えた理由や HWレイトレーシングを持つ奇特な *"RDNA" APU* を今になってサポートを進める理由は相変わらず不明。  
*NAVI10_LITE* もディスプレイコントローラに DCN 2.0.1 を持ち、今回のパッチはそれをドライバ部で認識することを目的としている。  

 > 		 	case FAMILY_NV:
 > 		 		dc_version = DCN_VERSION_2_0;
 > 		-		if (asic_id.chip_id == DEVICE_ID_NV_13FE) {
 > 		+
 > 		+		if (asic_id.chip_id == DEVICE_ID_NV_13FE || asic_id.chip_id == DEVICE_ID_NV_NAVI10_LITE_P_1400) {
 > 		 			dc_version = DCN_VERSION_2_01;
 > 		 			break;
 > 		 		}
 >
 > {{< quote >}} [[PATCH 13/24] drm/amd/display: Basic support with device ID](https://lists.freedesktop.org/archives/amd-gfx/2022-January/074104.html) {{< /quote >}}
