---
title: "Intel CPUのエラッタJCC、脆弱性TAAが公表される"
date: 2019-11-14T16:26:23+09:00
draft: true
tags: [ "Errata", "Vulnerability" ]
keywords: [ "Intel", "JCC", "Errata", "TAA", "Vulnerability"  ]
categories: [ "Hardware", "Intel", "CPU", "Security" ]
---

2019-11-12、Intel CPUのエラッタJCC、脆弱性Zombieload亜種のTAAが公表された。  
Intelは既に対策を施したマイクロコードを配布しており、Linux Kernel、Windowsも対応を進めている。  

### JCC (Jump Conditional Code)
[Benchmarks Of JCC Erratum: A New Intel CPU Bug With Performance Implications On Skylake Through Cascade Lake - Phoronix](https://www.phoronix.com/scan.php?page=article&item=intel-jcc-microcode&num=1)  
命令を格納するICache（Decoded Streaming Buffer、またはDSBとも呼ばれる）周りにエラッタがあり、  
Jump命令がキャッシュライン32Byteの境界を越えるとエラーが引き起こされるとのこと。  
マイクロコードの更新で、Jump命令が32Byteの境界を越える、または境界で終わる際、ICacheに格納されるのを防がられるようになる。  
ただこれはICacheミスの増加、レガシーデコードパイプラインへの切り替えにより性能低下を引き起こしてしまう。  
Intelは0〜4%だと言うが、上のPhoronixによるベンチマークでは明らかに4%では済まない結果が見られる。  
またIntelは更新したツールチェインで再コンパイルすることで、性能低下をある程度抑えられると言っているが、  
確かにマイクロコード更新前の性能に回復しているものもあれば、更新後と変わらなかったりかえって低下していたりと一概には言えない様子だ。  
Spectre/Meltdown/L1TF/MDSは、ユーザーとカーネルスペースを切り替えるワークロードにおいて性能低下の影響があったが、  
JCCは広範囲のワークロードに影響がある。  

影響を受けるCPUはSkylake世代からCascade Lake世代まで。  

 * Skylake
 * Kabylake
 * Coffee Lake
 * Whiskey Lake
 * Amber Lake
 * Cascade Lake

(追記)
[intel-microcode (3.20191112.1)  Changelog - Debian](https://metadata.ftp-master.debian.org/changelogs//non-free/i/intel-microcode/intel-microcode_3.20191112.1_changelog)  
DebianでのChangelogでは通常0〜3%、最悪10%と書かれていた。  
(追記終了)

### TAA (TSX Asynchronous Abort) <br> CVE-2019-11135
[Linux Kernel Gets Mitigations For TSX Async Abort Plus Another New Issue: iITLB Multihit - Phoronix](https://www.phoronix.com/scan.php?page=news_item&px=iITLB-Multihit-TAA-Kernel-Code)  

詳細な解説は無理なので諦める。  

[TAA - TSX Asynchronous Abort - Kernel.org](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/Documentation/admin-guide/hw-vuln/tsx_async_abort.rst?id=a7a248c593e4fd7a67c50b5f5318fe42a0db335e)  
[TSX Async Abort (TAA) mitigation - Kernel.org](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/Documentation/x86/tsx_async_abort.rst?id=a7a248c593e4fd7a67c50b5f5318fe42a0db335e)  

TSXのトランザクション処理のためのバッファが読み取れてしまう、ということなのだろうか？  
マイクロアーキテクチャによるもので、同じ公開範囲ということでMDS、Zombieloadの亜種とされているらしい。  
バッファはハイパースレッド間で共有される可能性があるということで、HTT（ハイパースレッティング）を無効化することで、ある程度緩和できる。  
対策としては、TSXを無効化することが望ましいとのこと。  
Linux Kernelでは対策として、デフォルトで[TSXを無効化&MDS対策を有効](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/Documentation/admin-guide/hw-vuln/tsx_async_abort.rst?id=a7a248c593e4fd7a67c50b5f5318fe42a0db335e#n274)することとなった。  
RHELでは[TSXは有効&MDS対策を有効](https://access.redhat.com/solutions/tsx-asynchronousabort)にするとのこと。  
Windowsでは直近のセキュリティアップデートで対策したとのことだが、TSXの有効/無効がどうなっているかは不明。  
[November 2019 Security Updates ](https://portal.msrc.microsoft.com/en-us/security-guidance/releasenotedetail/164aa83e-499c-e911-a994-000d3a33c573)

TSXと言えばHaswell世代から導入された新機能であり、メモリーロック処理とトランザクション機能をハードウェアで高速におこなうことで、  
最大6倍もの処理能力が謳われていたが、  
Haswell初期においてエラッタがあり、無効化され普及にブレーキがかかった。  
その後もあるモデルでは有効にされたり、無効にされたりとしていたが、（例: [i5-9400F](https://ark.intel.com/content/www/us/en/ark/products/190883/intel-core-i5-9400f-processor-9m-cache-up-to-4-10-ghz.html) i5-9400では有効になっている）  
今回の件ではブレーキどころか後退することとなるかもしれない。  
