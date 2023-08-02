---
title: "AMD MI300A と MI300C の CPUID Models"
date: 2023-08-03T00:45:31+09:00
draft: false
categories: [ "Hardware", "AMD", "APU", "CPU" ]
tags: [ "CDNA_3", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

AMD の Muralidhara M K 氏により、Linux の EDAC (Error Detection And Correction) ドライバーに **MI300A** と **MI300C** のサポートを追加するパッチが投稿されている。  
EDAC ドライバーはハードウェアが持つ MCE (Machine Check Exceptions) をデコードし、ハードウェアに起きたエラーの情報を読みやすい形式にして出力する機能を持つ。  

 * [[PATCH 0/7] AMD Family 19h Models 90h-9fh EDAC Support - Muralidhara M K](https://lore.kernel.org/linux-edac/20230720125425.3735538-1-muralimk@amd.com/)

**AMD MI300 シリーズ** は *Zen 4* CPU チップレットと *CDNA 3* GPU チップレットを組み合わせた HPC、AI 処理向けの製品ファミリー。  
**MI300A** は CPU チップレットと GPU チップレットを組み合わせた APU の形を取る SKU、**MI300X** は GPU チップレットのみで構成された SKU となる。  
パッチ内で触れられている **MI300C** はまだ正式発表されていない SKU だが、CPUID Model が設定されていることから少なくとも CPU チップレットを持った SKU と思われる。  

同じく AMD の Yazen Ghannam 氏によれば、**MI300A** は *Family 19h Models 90h-9Fh*、**MI300C** は *Family 19h Models 80h-8Fh* が CPUID Models に割り当てられている。  

 >         I suggest using MI300 in the $SUBJECT. And highlight in the message that
 >         both groups of MI300 use the same IDs. You can clarify that Models
 >         80h-8Fh are MI300C and 90h-9Fh are MI300A.
 >
 > {{< quote >}} [Re: [PATCH 1/7] x86/amd_nb: Add AMD Family 19h Models(80h-80fh) and (90h-9fh) PCI IDs - Yazen Ghannam](https://lore.kernel.org/linux-edac/6bce6a44-890e-9d50-dd10-e621b4d2a735@amd.com/) {{< /quote >}}

パッチでは、**MI300A (Family 19h Models 90h-9Fh)** の場合はメモリタイプを HBM3 と判定し、HBM3 コントローラーは 4基としているが、**MI300C (Family 19h Models 80h-8Fh)** に関するコードは含まれておらず、Root Complex や Data Fabric の PCI Device ID が追加されているのみとなる。  
そのため、**MI300C** のメモリタイプやメモリコントローラ数は不明となっている。  

 >         -	return 0x50000 + (umc << 20) + ((channel % 4) << 12);
 >         +	return gpu_umc_base + (umc << 20) + ((channel % 4) << 12);
 >         +}
 >         +
 >         +static void gpu_determine_memory_type(struct amd64_pvt *pvt)
 >         +{
 >         +	if (pvt->fam == 0x19) {
 >         +		switch (pvt->model) {
 >         +		case 0x30 ... 0x3F:
 >         +			pvt->dram_type = MEM_HBM2;
 >         +			break;
 >         +		case 0x90 ... 0x9F:
 >         +			pvt->dram_type = MEM_HBM3;
 >         +			break;
 >         +		default:
 >         +			break;
 >         +		}
 >         +	}
 >         +	edac_dbg(1, "  MEM type: %s\n", edac_mem_types[pvt->dram_type]);
 >
 > {{< quote >}} [[PATCH 5/7] EDAC/amd64: Add Fam19h Model 90h ~ 9fh enumeration support - Muralidhara M K](https://lore.kernel.org/linux-edac/20230720125425.3735538-6-muralimk@amd.com/) {{< /quote >}}
