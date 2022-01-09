---
title: "Intel Alchemist/DG2 は AV1エンコードをサポートか"
date: 2021-12-19T13:52:20+09:00
draft: false
tags: [ "Xe-HP", "Xe-HPG", "DG2", "Xe-HPC" ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "GPU" ]
noindex: false
# summary: ""
---


Intel GPU のメディア関連機能 (動画エンコード/デコード、ポストプロセッシング) をサポートする VA-API (Video Acceleration API) 向けドライバー、[Intel(R) Media Driver](https://github.com/intel/media-driver) で *Alchemist/DG2* 、*Ponte Vecchio* 、*{{< xe class="hp" >}} SDV/ATS-M (Arctic Sound-M?)* のサポートが進められている。  
そして masterブランチに取り込まれた変更の中に、*Alchemist/DG2* では AV1コーディックの HWエンコードに対応していることを示唆するものがあった。  

 * [Upstream Alchemist & ATS-M · intel/media-driver@244ccc6](https://github.com/intel/media-driver/commit/244ccc6f9e589f32a8789de4e5d46386faad6ea3)

以下が該当部となるが、エンコード機能に関する部分は条件付きコンパイル (`#ifdef IGFX_DG2_ENABLE_NON_UPSTREAM`) で制御されており、記述はあるが追加されていないファイルもあり、まだ media-driver に完全なサポートは追加されていないものと思われる。  

 > 		VAStatus MediaLibvaCapsDG2::LoadAv1EncProfileEntrypoints()
 > 		{
 > 		    VAStatus status = VA_STATUS_SUCCESS;
 > 		#ifdef IGFX_DG2_ENABLE_NON_UPSTREAM
 > 		#ifdef _AV1_ENCODE_VDENC_SUPPORTED
 > 		    AttribMap *attributeList = nullptr;
 > 		    if (MEDIA_IS_SKU(&(m_mediaCtx->SkuTable), FtrEncodeAV1Vdenc)||
 > 		        MEDIA_IS_SKU(&(m_mediaCtx->SkuTable), FtrEncodeAV1Vdenc10bit420))
 > 		    {
 > 		        status = CreateEncAttributes(VAProfileAV1Profile0, VAEntrypointEncSliceLP, &attributeList);
 > 		        DDI_CHK_RET(status, "Failed to initialize Caps!");
 >
 > {{< quote >}} [media-driver/media_libva_caps_dg2.cpp at 244ccc6f9e589f32a8789de4e5d46386faad6ea3 · intel/media-driver](https://github.com/intel/media-driver/blob/244ccc6f9e589f32a8789de4e5d46386faad6ea3/media_driver/linux/Xe_M/ddi/media_libva_caps_dg2.cpp#L41-L51) {{< /quote >}}
 > 
 > 		@@ VAStatus MediaLibvaCapsDG2::CheckEncodeResolution(
 >
 > 		        case VAProfileAV1Profile0:
 > 		        case VAProfileAV1Profile1:
 > 		             if ((width > CODEC_8K_MAX_PIC_WIDTH) ||
 > 		                 (width < m_encMinWidth) ||
 > 		                 (height > CODEC_8K_MAX_PIC_HEIGHT) ||
 > 		                 (height < m_encMinHeight))
 > 		             {
 > 		                 return VA_STATUS_ERROR_RESOLUTION_NOT_SUPPORTED;
 > 		             }
 > 		             break;
 > 
 > {{< quote >}} [media-driver/media_libva_caps_dg2.cpp at 244ccc6f9e589f32a8789de4e5d46386faad6ea3 · intel/media-driver](https://github.com/intel/media-driver/blob/244ccc6f9e589f32a8789de4e5d46386faad6ea3/media_driver/linux/Xe_M/ddi/media_libva_caps_dg2.cpp#L41-L51) {{< /quote >}}

あくまでも現時点での情報となるが、*Alchemist/DG2* は 8192x8192 までの AV1エンコードに対応しているように読み取れる。  
また、*Ponte Vecchio* 、*{{< xe class="hp" >}} SDV* 向けにも同様のファイルが追加されているが、AV1エンコードに関する記述は無い。この世代では *Alchemist/DG2* のみのサポートとなるのだろうか？  

## Alchemist/DG2・PVC L3Cache {#l3c}

L3キャッシュに関する記述も追加されている。  
*Alchemist/DG2* ではバンクあたり 512KB、Wayあたり 4KB、セクションあたり 2-Way の構成を採る。*Ponte Vecchio (PVC)* も同様の構成とされている。[^pvc-l3config]  

 > 		#define DG2_L3_CONFIG_COUNT     6
 > 		// 4KB per Way for DG2, two Way per section
 > 		static const L3ConfigRegisterValues DG2_L3_PLANES[DG2_L3_CONFIG_COUNT] =
 > 		{                                    //  Rest  R/W  RO   UTC  CB  Sum (in KB)
 > 		    {0x00000200, 0, 0, 0},           //  512   0    0    0    0   512
 > 		    {0xC0000000, 0x40000000, 0, 0},  //  384   0    0    128  0   512
 > 		    {0xF0000000, 0x00000080, 0, 0},  //  480   0    0    0    32  512
 > 		    {0x80000000, 0x80000000, 0, 0},  //  256   0    0    256  0   512
 > 		    {0x40000000, 0x00000080, 0, 0},  //  0     128  352  0    32  512
 > 		    {0x80000000, 0x70000080, 0, 0},  //  256   0    0    224  32  512
 > 		};
 >
 > {{< quote >}} [media-driver/media_interfaces_dg2.h at master · intel/media-driver](https://github.com/intel/media-driver/blob/master/media_driver/media_interface/media_interfaces_dg2/media_interfaces_dg2.h#L332-L342) {{< /quote >}}

[^pvc-l3config]: [media-driver/media_interfaces_pvc.h at 244ccc6f9e589f32a8789de4e5d46386faad6ea3 · intel/media-driver](https://github.com/intel/media-driver/blob/244ccc6f9e589f32a8789de4e5d46386faad6ea3/media_driver/media_interface/media_interfaces_pvc/media_interfaces_pvc.h#L317-L327)

*Tiger Lake* 等 *{{< xe class="lp" >}}* ではバンクあたり 480KB、Way あたり 4KB、セクションあたり 2-Way、  
*DG1* ではバンクあたり 2048KB (2MB)、Way あたり 16KB、セクションあたり 2-Way と構成。  

DGPU としての比較では、*Alchemist/DG2, Ponte Vecchio* の L3キャッシュは *DG1* より、バンクあたりの規模で小さいと言えるが、L3キャッシュ帯域ではバンク数が多い方が有利と考えられる。  
ローカルメモリを持つ dGPU になったことで、*{{< xe class="lp" >}}* からメモリバス幅が増えたことも関係するだろう。  

Intel GPU では L3キャッシュバンク内部を、データキャッシュやリードオンリーキャッシュ、Unified Tile Cache (UTC)、Command Buffer (CB) といった用途ごとに分けて割り当てる方式を採っている。  
*DG1* では L3キャッシュバンクの割り当て設定は 3パターンだったが、*Alchemist/DG2, PVC* では 6パターンであるため、より用途に合わせた割り当てが可能となっている。  
それと、*DG1* は内蔵 GPU を想定した *{{< xe class="lp" >}}* から構成をほとんど変えずに L3キャッシュを増やした結果、バンクあたり 2048KB、全体では 16MB というサイズになったと考えられる。  

 > 		// 16KB per Way for DG1, two Way per section
 > 		static const L3ConfigRegisterValues DG1_L3_PLANES[DG1_L3_CONFIG_COUNT] =
 > 		{                                      //  Rest  DC    RO  Z    Color  UTC  CB  Sum (in KB)
 > 		    {0x00000200, 0,          0, 0},    //  2048  0     0   0    0      0    0   2048
 > 		    {0x80000000, 0x7C000020, 0, 0},    //  1024  0     0   0    0      992  32  2048
 > 		    {0x0101F000, 0x00000020, 0, 0},    //  0     1024  992 0    0      0    32  2048
 > 		};
 >
 > {{< quote >}} [media-driver/cm_rt_g12_dg1.h at c2ab2faf51d766840d8d2f3a8b5c1c716d993b3f · intel/media-driver](https://github.com/intel/media-driver/blob/c2ab2faf51d766840d8d2f3a8b5c1c716d993b3f/cmrtlib/agnostic/share/cm_rt_g12_dg1.h#L35-L41) {{< /quote >}}

| L3$ | {{< xe class="lp" >}}<br>(TGL/RKL/ADL) | DG1 | {{< xe class="hpg/hpc" >}}<br>(Alchemist/DG2, PVC) |
| :-- | :--: | :--: | :--: |
| Up to L3$ size | 3840KB | 16384KB (16MB) | ? |
| L3$ size per bank | 480KB | 2048KB | 512KB |
| size per way | 4KB | 16KB | 4KB |
| way count per section | 2-Way | 2-Way | 2-Way |

{{< ref >}}
 * [Intel® Iris® Xe MAX Graphics Open Source Programmer's Reference Manual For the 2020 Discrete GPU formerly named "DG1" Volume 7: Memory Cache - intel-gfx-prm-osrc-dg1-vol07-memory_cache.pdf](https://01.org/sites/default/files/documentation/intel-gfx-prm-osrc-dg1-vol07-memory_cache.pdf)
 * [Volume 7: Memory Cache - intel-gfx-prm-osrc-tgl-vol07-memory_cache_0.pdf](https://01.org/sites/default/files/documentation/intel-gfx-prm-osrc-tgl-vol07-memory_cache_0.pdf)
 * [Intel Media For Linux | 01.org](https://01.org/intel-media-for-linux)
{{< /ref >}}
