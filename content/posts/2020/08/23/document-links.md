---
title: "Document Links"
date: 2020-08-23T04:10:40+09:00
draft: false
# tags: [ "", ]
# keywords: [ "", ]
categories: [ "Database", ]
noindex: true
summary: " "
---

{{< pindex >}}
 * [AMD](#amd)
    * [Github](#amd_github)
    * [Freedesktop](#amd_freedesktop)
 * [Intel](#intel)
    * [Github](#intel_github)
    * [Freedesktop](#intel_freedesktop)
 * [Coreboot / Chromium](#coreboot)
{{< /pindex >}}

## AMD {#amd}

   * [Welcome to AMD ׀ High-Performance Processors and Graphics](https://www.amd.com/en/)
      * [Tech Docs | AMD](https://www.amd.com/en/support/tech-docs/)
      * [AMD Products Specifications | AMD](https://www.amd.com/en/products/specifications/)
         * [Processor Specifications | AMD](https://www.amd.com/en/products/specifications/processors/)
         * [AMD Radeon™ Graphics Cards Specifications | AMD](https://www.amd.com/en/products/specifications/graphics/)
         * [Professional Graphics Specifications | AMD](https://www.amd.com/en/products/specifications/professional-graphics/)
         * [Embedded Processor Specifications | AMD](https://www.amd.com/en/products/specifications/embedded)
      * [Developer Guides, Manuals & ISA Documents - AMD](https://developer.amd.com/resources/developer-guides-manuals/)
   * [Let's Build Everything - GPUOpen](https://gpuopen.com/)
      * [RDNA - GPUOpen](https://gpuopen.com/rdna/)
         * [RDNA_Shader_ISA.pdf](https://developer.amd.com/wp-content/resources/RDNA_Shader_ISA.pdf)
         * [Introduction - rdna-whitepaper.pdf](https://www.amd.com/system/files/documents/rdna-whitepaper.pdf)
         * [AMD PowerPoint- White Template - RDNA_Architecture_public.pdf](https://gpuopen.com/wp-content/uploads/2019/08/RDNA_Architecture_public.pdf)

### Github {#amd_github}

 * [AMD](https://github.com/amd/)
 * [AMD Compute Libraries](https://github.com/AMDComputeLibraries/)
 * [AMDESE](https://github.com/AMDESE)
 * GPUOpen
    * [GPUOpen Drivers](https://github.com/GPUOpen-Drivers/)
      * [pal/ndDevice.cpp at dev · GPUOpen-Drivers/pal](https://github.com/GPUOpen-Drivers/pal/blob/dev/src/core/os/nullDevice/ndDevice.cpp)
         * ASIC info
    * [GPUOpen Libraries & SDKs](https://github.com/GPUOpen-LibrariesAndSDKs/)
    * [GPUOpen-Tools](https://github.com/GPUOpen-Tools/)
      * [common_src_device_info/DeviceInfoUtils.cpp at master · GPUOpen-Tools/common_src_device_info](https://github.com/GPUOpen-Tools/common_src_device_info/blob/master/DeviceInfoUtils.cpp)
         * Card info
         * ASIC Type, DeviceID, RevID, HW Generation, APU(bool), CAL name, Marketing name
 * ROCm(Radeon Open Compute)
    * [ROCm Core Technology](https://github.com/RadeonOpenCompute/)
    * [ROCm Developer Tools](https://github.com/ROCm-Developer-Tools/)
    * [ROCm Software Platform](https://github.com/ROCmSoftwarePlatform/)
 * [llvm/llvm-project](https://github.com/llvm/llvm-project)

### Freedesktop {#amd_freedesktop}

 * [RadeonFeature](https://www.x.org/wiki/RadeonFeature/)
 * Linux Kernel (KMD)
   * [The amd-gfx Archives](https://lists.freedesktop.org/archives/amd-gfx/)
   * <https://cgit.freedesktop.org/~agd5f/linux/?h=amd-staging-drm-next>
   * [AMD X.Org drivers - Patchwork](https://patchwork.freedesktop.org/project/amd-xorg-ddx/patches/)
 * [Mesa3D (User Mode Driver)](https://gitlab.freedesktop.org/mesa/mesa)
   * RadeonSI (OpenGL)
   * RADV (Vulkan)
   * [src/amd/common/ac_gpu_info.c · master · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/blob/master/src/amd/common/ac_gpu_info.c)
   * [src/amd/llvm/ac_llvm_util.c · master · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/blob/master/src/amd/llvm/ac_llvm_util.c)
   * [src/amd/addrlib/src/amdgpu_asic_addr.h · master · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/blob/master/src/amd/addrlib/src/amdgpu_asic_addr.h)
 * [Tom St Denis / umr · GitLab](https://gitlab.freedesktop.org/tomstdenis/umr/)
   * User Mode Register Debugger for AMDGPU Hardware
* [Mesa / drm · GitLab](https://gitlab.freedesktop.org/mesa/drm)
   * [data/amdgpu.ids · master · Mesa / drm · GitLab](https://gitlab.freedesktop.org/mesa/drm/blob/master/data/amdgpu.ids)
      * device_id, revision_id, product_name

## Intel {#intel}

 * [Intel® Graphics for Linux* | 01.org](https://01.org/linuxgraphics)
    * [Programmer's Reference Manuals | 01.org](https://01.org/linuxgraphics/documentation)
 * [GDC 2019 Tech Sessions](https://www.intel.com/content/www/us/en/developer/articles/event-contest/gdc-2019-tech-sessions.html?wapkw=Intel%20GDC%20Gen11)
    * [Intel® Processor Graphics Gen11 Architecture](https://www.intel.com/content/dam/develop/external/us/en/documents/02-the-architecture-of-intel-processor-graphics-gen11-807276.pdf)
 * [Intel® Processor Graphics Xᵉ-LP API Developer and Optimization Guide](https://www.intel.com/content/www/us/en/developer/articles/guide/lp-api-developer-optimization-guide.html)
 * [Intel® 64 and IA-32 Architectures Software Developer Manuals](https://software.intel.com/content/www/us/en/develop/articles/intel-sdm.html)
   * [Intel® 64 and IA-32 Architectures Optimization Reference Manual](https://software.intel.com/content/www/us/en/develop/download/intel-64-and-ia-32-architectures-optimization-reference-manual.html)

### Github {#intel_github}

 * [Intel Corporation](https://github.com/intel)
   * [intel/media-driver](https://github.com/intel/media-driver)
      * [media-driver/media_driver/linux at master · intel/media-driver](https://github.com/intel/media-driver/tree/master/media_driver/linux)
         * GPU Spec
         * `https://github.com/intel/media-driver/blob/master/media_driver/linux/gen{GPU_GEN}/ddi/media_sysinfo_g{GPU_GEN}.cpp`
   * [intel/intel-graphics-compiler](https://github.com/intel/intel-graphics-compiler)
      * [intel-graphics-compiler/igfxfmid.h at master · intel/intel-graphics-compiler](https://github.com/intel/intel-graphics-compiler/blob/master/inc/common/igfxfmid.h)
         * Product, PCI ID
      * [media-driver/media_driver/media_interface at master · intel/media-driver](https://github.com/intel/media-driver/tree/master/media_driver/media_interface)
         * L3cache Bank Config
   * [intel/compute-runtime: Intel® Graphics Compute Runtime for oneAPI Level Zero and OpenCL™ Driver](https://github.com/intel/compute-runtime)
      * [compute-runtime/opencl/source at master · intel/compute-runtime](https://github.com/intel/compute-runtime/tree/master/opencl/source)
         * GPU Spec
         * `https://github.com/intel/compute-runtime/blob/master/opencl/source/gen{GPU_GEN}/hw_info_{GPU_NAME}.inl`
   * [intel/gmmlib](https://github.com/intel/gmmlib)
      * [gmmlib/igfxfmid.h at master · intel/gmmlib](https://github.com/intel/gmmlib/blob/master/Source/inc/common/igfxfmid.h)
         * GPU/PCH PCI ID
 * [Sound Open Firmware](https://github.com/thesofproject)

### Freedesktop {#intel_freedesktop}

 * Linux Kernel (KMD)
   * [The Intel-gfx Archives](https://lists.freedesktop.org/archives/intel-gfx/)
 * [Mesa3D (User Mode Driver)](https://gitlab.freedesktop.org/mesa/mesa)
   * ANV (Vulkan)
   * [include/pci_ids/i965_pci_ids.h · master · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/blob/master/include/pci_ids/i965_pci_ids.h)
      * 〜 Gen11 PCI ID
   * [include/pci_ids/iris_pci_ids.h · master · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/blob/master/include/pci_ids/iris_pci_ids.h)
      * Gen12 PCI ID
   * [src/intel/dev/gen_device_info.c · master · Mesa / mesa · GitLab](https://gitlab.freedesktop.org/mesa/mesa/blob/master/src/intel/dev/gen_device_info.c)
      * GPU Spec
      * Slice, Sub-Slice, EU, L3cache Bank

## Coreboot / Chromium {#coreboot}
 * [project:coreboot · Gerrit Code Review](https://review.coreboot.org/q/project:coreboot)
 * [project:amd_blobs · Gerrit Code Review](https://review.coreboot.org/q/project:amd_blobs)
 * <https://chromium-review.googlesource.com>
