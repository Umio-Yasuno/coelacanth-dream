---
title: "Alder Lake-M 関連のパッチが Coreboot に投稿される、ADL-M は ADL-P の I/O 縮小版か"
date: 2021-01-21T00:32:24+09:00
draft: false
tags: [ "Alder_Lake", ]
# keywords: [ "", ]
categories: [ "Hardware", "Intel", "CPU" ]
noindex: false
# summary: ""
toc: false
---

CPU は *Golden Cove (Core)* と *Gracemont (Atom)* のハイブリッド、GPU は *Gen12アーキテクチャ* となる *Alder Lake* にはバリアントが、*Alder Lake-S* 、*Alder Lake-P* 、*Alder Lake-P* の 3種類が存在する。  
*Alder Lake-S* はデスクトップ向け、*Alder Lake-P* はモバイル向けであることは判明していたが、*Alder Lake-M* については不明な部分が多い。  
それが、今回 Coreboot のレポジトリに投稿された新たなパッチから *Alder Lake-M* の詳細が一部だが明らかにされた。  

## Alder Lake-P の I/O 縮小版？

*Alder Lake-M* の CPUID が追加されており、*Alder Lake-P* と末尾 1桁以外一致することから、*Alder Lake-M* は *Alder Lake-P* と同じコアであることが推測される。  
この CPUID には、`x86_Family` 、`X86_Model` といった情報を意味している。末尾 1桁は `Stepping` となり、CPU について判断する上で他よりはそれほど大きい意味ではないと思っている。  
読み取り方とかは以下リンクを参照。  
{{< link >}} [Alder Lake-P を搭載する Chromebookボード 「Brya」、そして Alder Lake-M | Coelacanth's Dream](/posts/2020/12/02/adl-p-chromebook-board-adl-m/) {{< /link >}}

 >        #define CPUID_ALDERLAKE_S_A0	0x90670
 >        #define CPUID_ALDERLAKE_P_A0	0x906a0
 >        #define CPUID_ALDERLAKE_M_A0	0x906a1
 >
 > {{< quote >}} <https://review.coreboot.org/c/coreboot/+/49629/3/src/soc/intel/common/block/include/intelblocks/mp_init.h> {{< /quote >}}

しかし *Alder Lake-M* には、I/O の規模に *Alder Lake-P* との違いが存在し、それよりも小さいものとなっている。  
*Alder Lake-P* は CPU側に PCIe Gen5 8-Lane x1、PCIe Gen4 4-Lane x2 の計 3ポートを持つが、*Alder Lake* ではそれが 1ポートに縮小、あるいは制限されている。  
PCH側のポートも減らされており、12ポートから 10ポートになっている。  

 >        config MAX_PCH_ROOT_PORTS
 >        	int
 >        	default 10 if SOC_INTEL_ALDERLAKE_PCH_M
 >        	default 12
 >        config MAX_CPU_ROOT_PORTS
 >        	int
 >        	default 1 if SOC_INTEL_ALDERLAKE_PCH_M
 >        	default 3
 >        config MAX_ROOT_PORTS
 >        	int
 >        	default MAX_PCH_ROOT_PORTS
 >        config MAX_PCIE_CLOCKS
 >        	int
 >        	default 10 if SOC_INTEL_ALDERLAKE_PCH_M
 >        	default 12
 >
 > {{< quote >}} <https://review.coreboot.org/c/coreboot/+/49630/3/src/soc/intel/alderlake/Kconfig#135> {{< /quote >}}

PCIe 以外でも一部減らされており、SATA は *Alder Lake-S/P* が 6ポート持つのに対し *Alder Lake-M* は 3ポートに、UART (Universal Asynchronous Receiver/Transmitte, シリアル通信に用いられる) も 7基から 4基となる。[^adl_m-sata]  

[^adl_m-sata]: <https://review.coreboot.org/c/coreboot/+/49629/3/src/soc/intel/common/block/sata/sata.c> <br> <https://review.coreboot.org/c/coreboot/+/49629/3/src/soc/intel/common/block/uart/uart.c>

## Alder Lake-M は Atom系プロセッサの後継か {#adl_m-atom}

*Alder Lake-M* は、*Alder Lake-P* から I/O の規模が減らされたことで、Atom系プロセッサに近づいている。  
ハイブリッドコア構成であることを考えれば、*Lakefield* に近いとも見られる。  
テクニカルライター 大原雄介 氏の記事によれば、Atomコアのみで構成されたプロセッサは *Elkhart/Jasper Lake* で最後となり、今後は Core系コアと組み合わせる形となるらしい。[^ascii]  
{{< comple >}} CPU構成はまだ明らかにされていないが {{< /comple >}} I/O 規模を見れば、*Alder Lake-M* はその後継に当てはまるように思える。  
そうなると、*Alder Lake-M* は CPU側の PCIe が 1ポートのみとなるが、その有効化されるポートが Gen5 8-Lane ではなく、Gen4 4-Lane になる可能性もある。  

[^ascii]: [ASCII.jp：最後のAtomとなるChromebook向けプロセッサーのJasper Lake　インテル CPUロードマップ (3/3)](https://ascii.jp/elem/000/004/040/4040489/3/)

*Alder Lake-P* と同じとすると GPU部が Gen12 96EU となり、多いように思えるが、電力効率の点では多数のユニットを低いクロックで動作させる方が良く、  
SDP (Scenario Design Power, 実使用を想定した消費電力) 7W の *Lakefield* 、**Core i5-L16G7** は Gen11 64EU を通常 200MHz、最大でも 500MHz の設定となっている。[^lkf]  
[DG1](/tags/dg1) や [Tiger Lake](/tags/tiger_lake) を見るに、Gen12 GPU は最大 1.5GHz近くでの動作が可能だが、*Alder Lake-M* ではその 1/2 か 1/3 程度のクロックになることが考えられる。  

[^lkf]: [Intel® Core™ i5-L16G7 Processor (4M Cache, up to 3.0GHz) Product Specifications](https://ark.intel.com/content/www/us/en/ark/products/202777/intel-core-i5-l16g7-processor-4m-cache-up-to-3-0ghz.html?wapkw=Lakefield)

Atomプロセッサは *Elkhart/Jasper Lake* で最後となるのは寂しく感じるが、*Alder Lake-M* がその後継となれば、ハイブリッドコア構成を採ることの恩恵の 1つであるシングルスレッド性能の大きな向上が受けられるのは喜ばしいことだろう。  
Linux や Coreboot での *Lakefield* サポートがそう積極的でなかったことから、ハイブリットプロセッサのサポートが *Alder Lake* の世代が本格的に行われていることも個人的にはかなり嬉しい。  

|     | Tiger Lake | Tiger Lake-H | Jasper Lake | Alder Lake-P | Alder Lake-M | Alder Lake-S |
| :-- | :--:       | :--:         | :--:        | :--:         | :--:         | :--: |
| GPU | Gen12 96EU | Gen12 32EU | Gen11 32EU | Gen12 96EU | Gen12 96EU? | Gen12 32EU |
| CPU side PCIe | Gen4<br>4-Lane x2 | Gen4<br> 16-Lane + 4-Lane |  | Gen5 8-Lanea,<br>Gen4 4-Lane x2 | Gen5 8-Lane? or<br>Gen4 4-Lane? | Gen5 8-Lane x2,<br>Gen4 4-Lane x2 |
| PCH | TGP_LP | TGP_H | JSP | ADP_P | ADP_M | ADP_S |
| PCH PCIe Ports | 16 | ? | 8 | 12 | 10 | 28 |



