---
title: "Linux 環境上で AMD GPU の情報を確認する方法"
date: 2022-12-18T05:17:56+09:00
draft: false
categories: [ "Software", "AMD", "GPU" ]
tags: [ "RadeonSI", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

タイトル通り、Linux 環境上で AMD GPU の各種スペックや ID、チップリビジョンといった情報を確認する方法。  
話をさっさと進めると、RadeonSI ドライバーのデバッグ情報を出力する方法が一番手軽かつ多くの情報が得られると思っている。他にアプリケーションをインストールする必要もない。  
一応書いておくと、RadeonSI ドライバーは Mesa3D ライブラリの AMD GPU 向け OpenGL ドライバーであり、最小構成の環境でなければインストールされているように思う。  
各種ファームウェアのバージョン情報等も出力されるため、バグを報告する際にも役立つ。  

`rocminfo` といったコマンドもあるが、RadeonSI ドライバーより得られる情報は少ない上、どちらかと言えば ROCm 環境が正常にインストールされているかを確かめるためのコマンドであるため、今回の目的には合わないと考えている。  

RadeonSI ドライバーのデバッグ情報を出力するには、環境変数 `AMD_DEBUG=info` を付けて OpenGL API を呼ぶアプリケーション、コマンドを実行すればいい。  
個人的には `glxinfo -B` がオススメで、OpenGL API を呼びながら新たにウィンドウを生成せず、コマンド自体が出力するメッセージも短めだ。  
最終的なコマンドは `AMD_DEBUG=info glxinfo -B` となる。  
一応書いておくと、`RADV_DEBUG=info` を付けて Vulkan API を呼ぶコマンドを実行しても同様の情報が出力される。  

 * [Environment Variables — The Mesa 3D Graphics Library latest documentation](https://docs.mesa3d.org/envvars.html#radeonsi-driver-environment-variables)

結果としては、記事下部のようなデバッグ情報が出力される。  
また、デバッグ情報を出力する部分のソースファイルは [src/amd/common/ac_gpu_info.c · main · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/blob/main/src/amd/common/ac_gpu_info.c) となる。  

`num_se` は有効な ShaderEngine 数、`num_rb` は RB (RenderBackend) 数、`num_cu` は CU 数を示す。  
CU 数やピーク GPUクロック (`max_gpu_freq`) から求められるピーク FP32 演算性能 (`max_gflops`)、メモリ速度 (`memory_freq`) やメモリバス幅 (`memory_bus_width`) から求められるピークメモリ帯域 (`memory_bandwidth`) といった情報も出力される。  

項目 `Identification` の `family, gfx_level, family_id, chip_external_rev` は主にドライバー中で必要とする値をそのまま出力しているため、ユーザー側で必要になる機会はほとんどない。  
`chip_rev` はチップリビジョンを示す値となる。  
`all_vram_visible` は Resizable BAR、Smart Access Memory が有効かを示すフラグ。`smart_access_memory` は `all_vram_visible` が有効な環境向けの最適化が有効かを示すフラグとなっている。  

 >         name of display: :0.0
 >         Device info:
 >             name = POLARIS11
 >             marketing_name = AMD Radeon RX 560 Series
 >             num_se = 2
 >             num_rb = 4
 >             num_cu = 16
 >             max_gpu_freq = 1196 MHz
 >             max_gflops = 2449 GFLOPS
 >             l1_cache_size = 16 KB
 >             l2_cache_size = 1024 KB
 >             memory_channels = 4 (TCC blocks)
 >             memory_size = 4 GB (4096 MB)
 >             memory_freq = 7 GHz
 >             memory_bus_width = 128 bits
 >             memory_bandwidth = 112 GB/s
 >             clock_crystal_freq = 25000 KHz
 >             IP GFX      8.0 	queues:1
 >             IP COMP     8.0 	queues:4
 >             IP SDMA     3.1 	queues:2
 >             IP UVD      6.3 	queues:1
 >             IP VCE      3.4 	queues:1
 >             IP UVD_ENC  6.3 	queues:1
 >         Identification:
 >             pci (domain:bus:dev.func): 0000:01:00.0
 >             pci_id = 0x67ff
 >             pci_rev_id = 0xcf
 >             family = 64
 >             gfx_level = 10
 >             family_id = 130
 >             chip_external_rev = 91
 >             chip_rev = 1
 >         Flags:
 >             is_pro_graphics = 0
 >             has_graphics = 1
 >             has_clear_state = 1
 >             has_distributed_tess = 1
 >             has_dcc_constant_encode = 0
 >             has_rbplus = 0
 >             rbplus_allowed = 0
 >             has_load_ctx_reg_pkt = 1
 >             has_out_of_order_rast = 1
 >             cpdma_prefetch_writes_memory = 1
 >             has_gfx9_scissor_bug = 0
 >             has_tc_compat_zrange_bug = 1
 >             has_msaa_sample_loc_bug = 1
 >             has_ls_vgpr_init_bug = 0
 >             has_32bit_predication = 0
 >             has_3d_cube_border_color_mipmap = 1
 >             never_stop_sq_perf_counters = 0
 >             has_sqtt_rb_harvest_bug = 0
 >             has_sqtt_auto_flush_mode_bug = 0
 >             never_send_perfcounter_stop = 0
 >             discardable_allows_big_page = 0
 >         Display features:
 >             use_display_dcc_unaligned = 0
 >             use_display_dcc_with_retile_blit = 0
 >         Memory info:
 >             pte_fragment_size = 2097152
 >             gart_page_size = 4096
 >             gart_size = 7938 MB
 >             vram_size = 4096 MB
 >             vram_vis_size = 4096 MB
 >             vram_type = 5
 >             max_heap_size_kb = 4096 MB
 >             min_alloc_size = 256
 >             address32_hi = 0x0
 >             has_dedicated_vram = 1
 >             all_vram_visible = 1
 >             smart_access_memory = 0
 >             max_tcc_blocks = 4
 >             tcc_cache_line_size = 64
 >             tcc_rb_non_coherent = 0
 >             pc_lines = 0
 >             lds_size_per_workgroup = 65536
 >             lds_alloc_granularity = 512
 >             lds_encode_granularity = 512
 >             max_memory_clock = 1750 MHz
 >         CP info:
 >             gfx_ib_pad_with_type2 = 0
 >             ib_alignment = 1024
 >             me_fw_version = 167
 >             me_fw_feature = 49
 >             mec_fw_version = 730
 >             mec_fw_feature = 49
 >             pfp_fw_version = 254
 >             pfp_fw_feature = 49
 >         Multimedia info:
 >             vce_encode = 1
 >             vcn_decode = 0
 >             vcn_encode = 0
 >             uvd_fw_version = 25300992
 >             vce_fw_version = 890897152
 >             vce_harvest_config = 2
 >         Kernel & winsys capabilities:
 >             drm = 3.48.0
 >             has_userptr = 1
 >             has_syncobj = 1
 >             has_timeline_syncobj = 1
 >             has_fence_to_handle = 1
 >             has_local_buffers = 0
 >             has_bo_metadata = 1
 >             has_eqaa_surface_allocator = 1
 >             has_sparse_vm_mappings = 1
 >             has_stable_pstate = 1
 >             has_scheduled_fence_dependency = 1
 >             mid_command_buffer_preemption_enabled = 0
 >             has_tmz_support = 0
 >         Shader core info:
 >             cu_mask[SE0][SA0] = 0xff 	(8)	CU_EN = 0xff
 >             cu_mask[SE1][SA0] = 0xff 	(8)	CU_EN = 0xff
 >             spi_cu_en_has_effect = 0
 >             max_good_cu_per_sa = 8
 >             min_good_cu_per_sa = 8
 >             max_se = 2
 >             max_sa_per_se = 1
 >             max_wave64_per_simd = 8
 >             num_physical_sgprs_per_simd = 800
 >             num_physical_wave64_vgprs_per_simd = 256
 >             num_simd_per_compute_unit = 4
 >             min_sgpr_alloc = 16
 >             max_sgpr_alloc = 104
 >             sgpr_alloc_granularity = 16
 >             min_wave64_vgpr_alloc = 4
 >             max_vgpr_alloc = 256
 >             wave64_vgpr_alloc_granularity = 4
 >             max_scratch_waves = 512
 >         Render backend info:
 >             pa_sc_tile_steering_override = 0x0
 >             max_render_backends = 4
 >             num_tile_pipes = 4
 >             pipe_interleave_bytes = 256
 >             enabled_rb_mask = 0xf
 >             max_alignment = 262144
 >             pbb_max_alloc_count = 0
 >         GB_ADDR_CONFIG: 0x22011002
 >             num_pipes = 4
 >             pipe_interleave_size = 256
 >             bank_interleave_size = 1
 >             num_shader_engines = 2
 >             shader_engine_tile_size = 32
 >             num_gpus = 0 (raw)
 >             multi_gpu_tile_size = 2 (raw)
 >             row_size = 4096
 >             num_lower_pipes = 0 (raw)
