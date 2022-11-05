---
title: "Intel SDSi の名称が Intel On Demand に"
date: 2022-11-06T05:30:02+09:00
draft: false
categories: [ "Hardware", "Intel", "CPU" ]
tags: [ "Sapphire_Rapids", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Intel は Xeon ファミリープロセッサでハードウェアがサポートする新機能、Intel SDSi (Software Defined Silicon) のソフトウェアサポートを進めているが、Intel SDSi の名称が Intel On Demand に置き換えられた。  
仕様書を公開している [intel/intel-sdsi](https://github.com/intel/intel-sdsi) レポジトリ、サポートを進めている Linux Kernel でその動きが見られる。  

 * [[PATCH 0/9] Extend Intel On Demand (SDSi) support - David E. Box](https://lore.kernel.org/lkml/20221101191023.4150315-1-david.e.box@linux.intel.com/)
 * [intel/intel-sdsi](https://github.com/intel/intel-sdsi)

## Intel On Demand {#ondemand}
Intel SDSi/On Demand は Xeon プロセッサに搭載されているが無効化されている機能またはアクセラレータを、後からライセンスを購入し、認証することで有効化するための機能となる。  
Intel が公開している Intel On Demand のページによれば、Consumption Model (Metered) と Activation Model (One-time) が用意されており、Activation Model の対象には SGX (Software Guard Extensions), QAT (Quick Assist Technology), Dynamic Load Balancer, DSA (Data Streaming Accelerator), IAA (In-Memory Analytics Accelerator) が含まれている。  
推測だが、SGX については SGX Enclave の有効化と専用メモリ領域の最大サイズがライセンスの対象になると思われる。  

 * [Intel On Demand](https://www.intel.com/content/www/us/en/products/docs/ondemand/overview.html)

Consumption Model (Metered)、従量制ライセンスは需要に合わせた性能と容量のスケーリングを提供するとしているが、具体的に何の機能やスペックが対象になるかは記載されていない。  

サポートする Xeon SKU がまだ正式発表されていないこともあり、詳細が不明な部分は多い。  
Intel SDSi/On Demand をサポートすると見られる第 4世代 Xeon ファミリープロセッサ (*Sapphire Rapids-SP*) は、2023/01/10 に発表イベントを開催することが Intel 公式より予告されている。[^spr-launch]  

[^spr-launch]: [Chalk Talk Covers Strategy and Design Behind 4th Gen Intel Xeon...](https://www.intel.com/content/www/us/en/newsroom/article/chalk-talk-strategy-design-new-xeon-processor.html)

Intel SDSi/On Demand の動作としては、Linux Kernel へのパッチや仕様を読むにライセンスの認証と適用はソケットごとに行われる。  
CPU は Intel SDSi/On Demand 用の NVRAM を持ち、Authentication Key Certificate (AKC) と Capability Activation Payload (CAP) を書き込むのに使われる。  
ハードウェアによる検証と認証が成功すると CPU 構成が更新される。機能の完全な有効化にはコールドリブートが必要としていることから、CPU 構成自体も NVRAM に書き込まれるのだろうか。  
また、Consumption Model (Metered) 向けの Counter Unit の存在も確認できる。  

 >         +		On Demand NVRAM for the CPU. CAPs are used to activate a given
 >         +		CPU feature. A CAP is validated by On Demand hardware using a
 >         +		previously provisioned AKC file. Upon successful authentication,
 >         +		the CPU configuration is updated. A cold reboot is required to
 >         +		fully activate the feature. Mailbox command.
 >         
 > {{< quote >}} [[PATCH 1/9] platform/x86/intel/sdsi: Add Intel On Demand text - David E. Box](https://lore.kernel.org/lkml/20221101191023.4150315-2-david.e.box@linux.intel.com/) {{< /quote >}}

繰り返すが Intel SDSi/On Demand にはまだ詳細が不明な部分が多い。  
あくまでもオプション機能を追加するためのもので、それとは別に最初から機能やアクセラレーターが有効化された SKU、言い換えれば買い切り型のライセンスを同梱した SKU も提供されるのか、といった点も不明である。  
Intel Xeon SKU は種類が豊富、少し悪く言えば複雑であり、SKU 間でコア数や対応ソケット数、SGX Enclave の最大サイズ、SST (Speed Select Technology) サポートの有無、Optane Persistent Memory サポートといった違いがある。  
第 2世代 Xeon SKU (*Cascade Lake-SP*) では、AVX512 命令の実行ポート数が 上位 SKU は 2ポート、下位 SKU は 1ポートといった違いもあった。  
Intel SDSi/On Demand は、世代を経るごとに追加される機能やアクセラレーターによって SKU がさらに複雑化するのを抑える助けになるかもしれないが、ライセンスモデルやライセンス管理によるコストがどれだけ受け入れられるかは不透明である。  

{{< ref >}}
 * [2nd Gen Intel® Xeon® Scalable Processors Brief](https://www.intel.co.jp/content/www/jp/ja/products/docs/processors/xeon/2nd-gen-xeon-scalable-processors-brief.html)
 * [第 3 世代インテル® Xeon® スケーラブル・プロセッサー・ファミリーの概要](https://www.intel.co.jp/content/www/jp/ja/products/docs/processors/xeon/3rd-gen-xeon-scalable-processors-brief.html)
{{< /ref >}}
