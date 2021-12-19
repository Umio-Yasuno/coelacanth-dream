---
title: "AMD Yellow Carp/Rembrandt APU ブートログ&emsp;―― LilacKD-RMB, 100-000000560-40_Y"
date: 2021-12-14T20:21:08+09:00
draft: false
tags: [ "Yellow_Carp", "RDNA_2" ]
# keywords: [ "", ]
categories: [ "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
---

[Ubuntu in Launchpad](https://launchpad.net/ubuntu) にて、You-Sheng Yang 氏によるバグ報告の中で *Yellow Carp/Rembrandt APU* の Linux Kernel ブートログが投稿されている。  

 * [Bug #1953008 “Support USB4 DP alt mode for AMD Yellow Carp graph...” : Bugs : linux-firmware package : Ubuntu](https://bugs.launchpad.net/ubuntu/+source/linux-firmware/+bug/1953008)

使用しているボードは *AMD LilacKD-RMB/LilacKD-RMB* 。  

 > 		[    0.000000] DMI: AMD LilacKD-RMB/LilacKD-RMB, BIOS RRL0080C 09/15/2021
 >
 > {{< quote >}} [](https://launchpadlibrarian.net/571858317/CurrentDmesg.txt) {{< /quote >}}

{{< pindex >}} 
 * [CPU](#cpu)
 * [GPU](#gpu)
 * [Memory](#memory)
{{< /pindex >}}

## CPU {#cpu}

プロセッサ名は **AMD Eng Sample: 100-000000560-40_Y** 、エンジニアサンプリング品となる。  
CPUID における `Family, Model` は `family: 0x19 (25), model: 0x44 (68)`、EAXレジスタそのままの値に書くと `0x00A40F40`。  
*Yellow Carp/Rembrandt* の `Family, Model` 情報は、AMD CPU の温度センサーを読み取る `k10temp` ドライバーにサポートを追加する際に、`Model 40h-4fh` となることが明かされていた。  
{{< link >}} [Yellow Carp は Family 19h Model 40h-4Fh | Coelacanth's Dream](/posts/2021/08/27/yc-x86-model/) {{< /link >}}

 > 		[    0.195807] smpboot: CPU0: AMD Eng Sample: 100-000000560-40_Y (family: 0x19, model: 0x44, stepping: 0x0)
 >
 > {{< quote >}} [](https://launchpadlibrarian.net/571858317/CurrentDmesg.txt) {{< /quote >}}

*Yellow Carp/Rembrandt* の TLBエントリ数は以下のようになっており、*Zen 3 アーキテクチャ* と同様の構成となっている。  

 > 		[    0.082550] Last level iTLB entries: 4KB 512, 2MB 512, 4MB 256
 > 		[    0.082551] Last level dTLB entries: 4KB 2048, 2MB 2048, 4MB 1024, 1GB 0
 >
 > {{< quote >}} [](https://launchpadlibrarian.net/571858317/CurrentDmesg.txt) {{< /quote >}}

CPUスレッド数は 16-Thread 認識されている。  
ブートログと一緒に投稿されている `/proc/cpuinfo` の情報では 8-Core/16-Thread とされている。  
対応する機能、命令も *Zen 3 アーキテクチャ* とほとんど同じものになっている。  
`FSRM (Fast Short REP MOV)` のフラグが立っていないが、BIOS/UEFI から無効化されているのかもしれない。自分が今使用している A520 M/B に `FSRM` の有無を切り替えるオプションが存在していた。  
CPUクロックは最小 1.6GHz、ブートログの情報より、ベース 3.1GHz (3094MHz) で動作している。  
L3キャッシュの情報は無いが、AMD GPUドライバーに記述されているセンサー情報から L3キャッシュは 1基、8-Core CCX 1基の構成とされる。[^smu13]  

[^smu13]: [linux/smu13_driver_if_yellow_carp.h at 385bb92fdc5813c5f6a8168d6bba8680f2c1d0de · torvalds/linux](https://github.com/torvalds/linux/blob/385bb92fdc5813c5f6a8168d6bba8680f2c1d0de/drivers/gpu/drm/amd/pm/inc/smu13_driver_if_yellow_carp.h#L173-L178)

 > 		processor	: 0
 > 		vendor_id	: AuthenticAMD
 > 		cpu family	: 25
 > 		model		: 68
 > 		model name	: AMD Eng Sample: 100-000000560-40_Y
 > 		stepping	: 0
 > 		microcode	: 0xa404002
 > 		cpu MHz		: 1600.000
 > 		cache size	: 512 KB
 > 		physical id	: 0
 > 		siblings	: 16
 > 		core id		: 0
 > 		cpu cores	: 8
 > 		apicid		: 0
 > 		initial apicid	: 0
 > 		fpu		: yes
 > 		fpu_exception	: yes
 > 		cpuid level	: 16
 > 		wp		: yes
 > 		flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx mmxext fxsr_opt pdpe1gb rdtscp lm constant_tsc rep_good nopl nonstop_tsc cpuid extd_apicid aperfmperf rapl pni pclmulqdq monitor ssse3 fma cx16 sse4_1 sse4_2 x2apic movbe popcnt aes xsave avx f16c rdrand lahf_lm cmp_legacy svm extapic cr8_legacy abm sse4a misalignsse 3dnowprefetch osvw ibs skinit wdt tce topoext perfctr_core perfctr_nb bpext perfctr_llc mwaitx cpb cat_l3 cdp_l3 hw_pstate ssbd mba ibrs ibpb stibp vmmcall fsgsbase bmi1 avx2 smep bmi2 invpcid cqm rdt_a rdseed adx smap clflushopt clwb sha_ni xsaveopt xsavec xgetbv1 xsaves cqm_llc cqm_occup_llc cqm_mbm_total cqm_mbm_local clzero irperf xsaveerptr rdpru wbnoinvd arat npt lbrv svm_lock nrip_save tsc_scale vmcb_clean flushbyasid decodeassists pausefilter pfthreshold avic v_vmsave_vmload vgif v_spec_ctrl umip pku ospke vaes vpclmulqdq rdpid overflow_recov succor smca
 > 		bugs		: sysret_ss_attrs spectre_v1 spectre_v2 spec_store_bypass
 > 		bogomips	: 6188.30
 > 		TLB size	: 2560 4K pages
 > 		clflush size	: 64
 > 		cache_alignment	: 64
 > 		address sizes	: 48 bits physical, 48 bits virtual
 > 		power management: ts ttp tm hwpstate cpb eff_freq_ro [13] [14]
 >
 > {{< quote >}} [](https://launchpadlibrarian.net/571858415/ProcCpuinfo.txt) {{< /quote >}}

## GPU {#gpu}

**AMD Eng Sample: 100-000000560-40_Y** の GPU部は DeviceID: 0x1681, RevID: 0xC7 。  
*Yellow Carp/Rembrandt* の DeviceID には既に 2つが割り当てられており、今回の 0x1681 ともう 1つ、0x164D がある。  
{{< link >}} [既に 2つの DeviceID が用意されている Yellow Carp APU ファミリー | Coelacanth's Dream](/posts/2021/07/26/yc-apu-two-did/) {{< /link >}}

 > 		[    1.746371] [drm] initializing kernel modesetting (YELLOW_CARP 0x1002:0x1681 0x1002:0x0123 0xC7).
 >
 > {{< quote >}} [](https://launchpadlibrarian.net/571858317/CurrentDmesg.txt) {{< /quote >}}
 >
 > 		[    1.755206] amdgpu 0000:62:00.0: amdgpu: Fetched VBIOS from VFCT
 > 		[    1.755208] amdgpu: ATOM BIOS: 113-REMBRANDT-032
 >
 > {{< quote >}} [](https://launchpadlibrarian.net/571858317/CurrentDmesg.txt) {{< /quote >}}

GPU部の構成は、SE (ShaderEngine) 1基、SE あたりの SH (ShaderArray) 2基、SH あたりの CU 6基で、総 CU 12基が有効化されている。  

 > 		[    3.876317] amdgpu: HMM registered 512MB device memory
 > 		[    3.876352] amdgpu: SRAT table not found
 > 		[    3.876354] amdgpu: Virtual CRAT table created for GPU
 > 		[    3.876394] amdgpu: Topology: Add dGPU node [0x1681:0x1002]
 > 		[    3.876398] kfd kfd: amdgpu: added device 1002:1681
 > 		[    3.876417] amdgpu 0000:62:00.0: amdgpu: SE 1, SH per SE 2, CU per SH 6, active_cu_number 12
 >
 > {{< quote >}} [](https://launchpadlibrarian.net/571858317/CurrentDmesg.txt) {{< /quote >}}

*Yellow Carp/Rembrandt* の GPUキャッシュ構成もまた、以前に (2021/06) 投稿された AMD GPUドライバーへのパッチで既に明かされている。  
{{< link >}} [RDNA 2 APU 「Yellow Carp」 をサポートするパッチが Linux Kernel に投稿される　―― DCN3.1 / VanGogh より大きいキャッシュ | Coelacanth's Dream](/posts/2021/06/03/yellow_carp-apu-linux-kernel/#yc-cache) {{< /link >}}

*Yellow Carp/Rembrandt* は GPU部に *RDNA 2 アーキテクチャ* を採用しており、同様の APU には *VanGogh (Aerith) APU* がいる。  
そして *Yellow Carp/Rembrandt* は *VanGogh* と比べて、CU数だけでなくキャッシュの規模も大きくしている。  
*RDNA系アーキテクチャ* では、SH ごとに GL1キャッシュ 128KiB を持ち、ほとんどの RDNA/2 GPU では SE あたり SH 2基という構成を採っているが、*VanGogh* は SE あたり SH 1基という GL1キャッシュを減らし、小規模であることを意識した構成だった。  
*Yellow Carp/Rembrandt* のキャッシュが大きいというより、*VanGogh* が特別小さかったと言った方が適切かもしれない。一応 *RDNA 2 APU* という枠内で強化された点ではある。  
また、GPU全体で共有する L2キャッシュも、*VanGogh* では 2MiB だったのが *Yellow Carp/Rembrandt* では 4MiB となり、容量を増やしている。  
なお現時点の *RDNA 2 APU* は共通して L3/Infinity Cache (MALL) を持たない。  

| RDNA 2 APU | VanGogh | Yellow Carp<br>(Rembrandt) |
| :-- | :--: | :--: |
| GL1 Cache | 128KiB | 256 KiB<br>(2x 128KiB)
| L2 Cache | 2MiB | 4MiB |
| L3 Cache/MALL | N/A | N/A |

## Memory {#memory}

メモリは DDR5 64-bit、メモリサイズは約 8GB となっている。  
AMD GPUドライバーは LPDDR5、DDR5 をまとめて DDR5 と認識し、また VBIOS からメモリチャネル数を読み取り、そこに 64 を掛けてメモリバス幅とする実装になっている。  
少し話がずれるが、以前このことを書いた時に出した VanGogh は LPDDR5 64-bit という推測は結局間違っていた。  
{{< link >}} [【雑記】 VanGogh APU のメモリインターフェイスは (恐らく) LPDDR5 64-bit | Coelacanth's Dream](/posts/2021/03/19/coelacanth-diary-2021-03-19/) {{< /link >}}
それとして、メモリサイズから今回のブートログは 1x DDR5 8GB 構成のものではないかと思われる。  

 > 		[    1.755245] [drm] Detected VRAM RAM=512M, BAR=512M
 > 		[    1.755246] [drm] RAM width 64bits DDR5
 > 		[    1.755269] [drm] amdgpu: 512M of VRAM memory ready
 > 		[    1.755270] [drm] amdgpu: 3072M of GTT memory ready.
 >
 > {{< quote >}} [](https://launchpadlibrarian.net/571858317/CurrentDmesg.txt) {{< /quote >}}
 >
 > 		[    0.038348] Memory: 7185464K/7640100K available (16393K kernel code, 3525K rwdata, 5504K rodata, 2904K init, 5712K bss, 454376K reserved, 0K cma-reserved)
 >
 > {{< quote >}} [](https://launchpadlibrarian.net/571858317/CurrentDmesg.txt) {{< /quote >}}
