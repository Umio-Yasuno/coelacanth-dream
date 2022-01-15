---
title: "ROCm OpenCL v4.5 は Polaris (gfx803) をサポートせず"
date: 2021-12-13T13:46:32+09:00
draft: false
tags: [ "ROCm", ]
# keywords: [ "", ]
categories: [ "Software", "AMD", "GPU" ]
noindex: false
# summary: ""
---

インストールガイドの変更と `roc-obj` ツール、OpenMPオフロードにおけるプロファイル/トレース機能の修正を含む ROCm v4.5.2 がリリースされた。  

 * [RadeonOpenCompute/ROCm at roc-4.5.x](https://github.com/RadeonOpenCompute/ROCm/tree/roc-4.5.x)

新リリースはそれとして、ROCm v4.5.x では OpenCLランタイムから *Polaris (gfx803)* のサポートが外されていることに気付いた。  
v4.3.\[0-1\] では正常に動作したのだが、v4.5.0 からはプラットフォーム自体は認識されるものの、*Polaris (gfx803)* がデバイスとして認識されなくなった。  

ROCm v4.5.0 リリース時に、一部 ROCmソフトウェア/ライブラリから *Polaris (gfx803)* のサポートを取り除く動きが見られることに触れたが、ROCmの各種ランタイム、内部コードも同様の作業を進めていることとなる。  
{{< link >}} [ROCm v4.5 がリリース ―― CPU+GPU の統合メモリをサポート、RDNA系のサポートは v5.0 に？ | Coelacanth's Dream](/posts/2021/10/30/rocm-4_5-release/#polaris-rdna) {{< /link >}}

このことは以下の issue でも報告されている。  
issue内のコメントにあるように、「公式的にはサポートしない」ことと「サポートを取り除く」ことには、意味としても実際の結果としても大きな開きがある。  

 * [RX 470 card no longer recognized by clinfo after 4.5 update · Issue #1608 · RadeonOpenCompute/ROCm](https://github.com/RadeonOpenCompute/ROCm/issues/1608)
    * <https://github.com/RadeonOpenCompute/ROCm/issues/1608#issuecomment-965733228>

*Polaris (gfx803)* は ROCm の初期バージョンからサポートされ、またその価格帯から広く出回った GPU であるため、該当 issue では ROCm v4.5.x での変更を残念に思うコメントが見られる。  
参考までに載せると、[Steamハードウェア＆ソフトウェア 調査](https://store.steampowered.com/hwsurvey/?platform=linux) によれば、Linuxプラットフォームにおいて 2021/11 で最も使用されている GPU は *Radeon RX 480 (Polaris)* であり、6.76% のシェアを持っている。  

ROCmSupportアカウントは、意図的な変更ではない可能性があるとしているが、*Polaris (gfx803)* を含む *GFX8* 世代の AMD GPU は ROCm v4.0.0 から公式サポートから外されており、検証が行われていないとしている。  
実質的にはサポートが削除され、今後戻されることはない、と言っているようにも聞こえる。  

 * <https://github.com/RadeonOpenCompute/ROCm/issues/1608#issuecomment-966159384> 

## ROCm v4.3.1 を再インストールする {#v4_3_x}

自環境で実践した ROCm v4.3.1 の再インストール方法。  
最近になって AMDGPU-PRO と ROCm のインストール方法が変わり、`amdgpu-install` パッケージのインストール時にレポジトリURLが追加され、そこから `amdgpu-install` コマンドで各種 AMDGPU-PRO/ROCm パッケージをインストールする形となった。  

 * [ROCm Installation Guide v4.5 — ROCm 4.5.0 documentation](https://rocmdocs.amd.com/en/latest/Installation_Guide/Installation_new.html#installation-methods)
 * <http://repo.radeon.com/amdgpu-install/>

最新バージョンの 21.40.2 で追加される ROCmレポジトリ (`rocm.list`) は、ROCm v4.5.2 のものであるため、`apt edit-sources rocm.list` 等で URL部末尾を `4.3.1` に書き換えればそのバージョンのパッケージをインストール可能になる。  
ROCm v4.5.x のパッケージを一度削除してから、v4.3.1 のを再インストールしないと反映されない場合があるため注意。  

 > 		# deb [arch=amd64] https://repo.radeon.com/rocm/apt/4.5.2 ubuntu main 
 > 		deb [arch=amd64] https://repo.radeon.com/rocm/apt/4.3.1 ubuntu main 

`amdgpu-install` コマンドには `--rocmrelease=` オプションが存在するが、自分が試した限り、別バージョンの ROCm をインストールしたい場合は手動でレポジトリを追加する必要があった。  
またヘルプメッセージには記載されていないが、`--rocmrelease=` は `<major>.<minor>.<step>` のフォーマットでバージョンを指定する。  

AMDGPU-PRO の OpenCL (Legacy) であれば最新バージョンでも *Polaris (gfx8)* をサポートしているため、そちらを使う手もある。  

 > 		Usage: amdgpu-install [options...]
 > 		
 > 		Options:
 > 		  -h|--help                Display this help message
 > 		  --dryrun                 Print list of commands to run and exit
 > 		  --pro                    (DEPRECATED) Install legacy OpenGL, pro Vulkan, and
 > 		                           open source multimedia. This is equivalent to:
 > 		                           amdgpu-install --usecase=workstation --vulkan=pro
 > 		  --usecase=               Install a set of libraries for a specific use case
 > 		  --list-usecase           Show all available usecases and descriptions
 > 		  --opencl=                Install a specific OpenCL implementation. This option
 > 		                           implies the addition of the opencl usecase.
 > 		                           Available implementations:
 > 		                           rocr        (ROCr/KFD based OpenCL)
 > 		                           legacy      (Legacy OpenCL)
 > 		  --vulkan=                Install a specific vulkan implementation
 > 		                           Available implementations:
 > 		                           amdvlk      (AMD open source implementation)
 > 		                           pro         (AMD closed source implementation)
 > 		  --no-dkms                Do not install dkms and use built-in kernel driver
 > 		  --no-32                  Do not install 32 bit graphics runtime support
 > 		  --rocmrelease=           Install a specific ROCm release. By default only
 > 		                           one release of ROCm can be installed. Using this
 > 		                           option will allow installation of multiple releases.
 > 		                           Note: when used during uninstall, the specific rocm
 > 		                                 release will be removed. Use --rocmrelease=all
 > 		                                 to uninstall all rocm releases.
 > 		  --accept-eula            Accept EULA for this run only (for non-free install)
 > 		                           Note: only use this option if you accept the EULA
 > 		  --uninstall              Uninstall the graphics driver
 > 		
 > 		  Options --opencl/--vulkan/--usecase can be used together, e.g.:
 > 		  amdgpu-install --usecase=graphics --vulkan=amdvlk --opencl=rocr
 > 		
 > 		  Multiple implementations can be selected if comma separated, e.g.:
 > 		  amdgpu-install --usecase=graphics,opencl --opencl=rocr,legacy --vulkan=amdvlk,pro
 > 		
 > 		  Unless the -h|--help option is given, 'apt-get' options may be present

