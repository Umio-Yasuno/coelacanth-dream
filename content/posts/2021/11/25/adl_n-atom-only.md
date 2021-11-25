---
title: "Adl_n Atom Only"
date: 2021-11-25T12:53:10+09:00
draft: true
tags: [ "", ]
# keywords: [ "", ]
categories: [ "", ]
noindex: false
# summary: ""
---

 * [soc/intel/alderlake, soc/common: Add method to determine the cpu type mask (If4ceb24d) · Gerrit Code Review](https://review.coreboot.org/c/coreboot/+/59360/3)

 > 		 +uint8_t get_soc_cpu_type(void)
 > 		+{
 > 		+	struct cpuinfo_x86 cpuinfo;
 > 		+
 > 		+	if (cpu_is_hybrid_supported())
 > 		+		return cpu_get_cpu_type();
 > 		+
 > 		+	get_fms(&cpuinfo, cpuid_eax(1));
 > 		+
 > 		+	if (cpuinfo.x86 == 0x6 && cpuinfo.x86_model == 0xBE)
 > 		+		return CPUID_CORE_TYPE_INTEL_ATOM;
 > 		+	else
 > 		+		return CPUID_CORE_TYPE_INTEL_CORE;
 > 		+}
 > 		+
 >
 > {{< quote >}} <https://review.coreboot.org/plugins/gitiles/coreboot/+/3352093f2bb1f6912dd57d19a5e564e8d21ecc26%5E%21/#F0> {{< /quote >}}
