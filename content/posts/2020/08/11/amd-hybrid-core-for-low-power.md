---
title: "AMD の省電力に向けたハイブリットコア技術を読む"
date: 2020-08-11T02:35:49+09:00
draft: false
# tags: [ "", ]
keywords: [ "", ]
categories: [ "Hardware", "AMD", "CPU" ]
noindex: false
# summary: ""
---

AMD はプロセッサの省電力動作のためのハイブリッドコア技術の特許を提出し、その情報を Twitterユーザーの [@Underfox3 / Twitter](https://twitter.com/Underfox3) 氏が共有、拡散した。  
その特許の概要を読み、自分の知識が及ぶ範囲で記事とする。  
{{< link >}} [Instruction subset implementation for low power operation](http://www.freepatentsonline.com/10698472.pdf) {{< /link >}}

堅い特許文書を自分なりに噛み砕いているため、誤解釈等が含まれている可能性があります。  
もし誤解釈があれば、気軽に連絡下さると嬉しいです。  

{{< pindex >}}

 * [概要](#summary)
    * [High-Feature, Low-Feature](#high-low-processor)
    * [実行コアの切り替え方法](#switch-exec-core)
    * [利点と欠点](#merit-demerit)
 * [余談](#aside)

{{< /pindex >}}


## 概要 {#summary}
まず、特許の内容としては、規模(性能、電力効率)、対応命令範囲の異なる 2種類の CPUコアに対し、
どのようにスレッドを実行する CPUコアを切り替えるか、というものとなっている。  

### High-Feature, Low-Feature {#high-low-processor}
文書では、2種類の CPUコアを `HIGH-FEATURE` 、`LOW-FEATURE` に分けており、この記事では便宜上 `High`コア/高機能コア、`Low`コア/低機能(省電力)コアとする。  

`High`コアと `Low`コアではコア構成が異なり、例えば、キャッシュメモリの容量、キャッシュ階層、分岐予測機構、実行ユニット、In-Order 実行か Out-Of-Order 実行か、投機的実行に対応するかしないか等の点で異なる。  
浮動小数点演算機能やテーブルウォーク(TLBミスが発生した際、メモリ上のページテーブルを走査し、エントリを入れ替える機能) を実装しない例も示されている。  
また、`High`コアは `Low`コアが対応する ISA 全てに対応するが、`Low`コアは `High`コアが対応する ISA 全てには対応しない。  


`High`コアは性能に優れるが、消費電力も大きく、専有するダイエリアも大きい。  
`Low`コアは性能こそ低く、対応する機能も少ないが、消費電力は小さく、電力効率に優れ、必要とするダイエリアも小さい。  

この特徴の異なる `High`コア、`Low`コアをワークロード、アプリケーションに応じて如何に切り替えるかが肝となる。  

### 実行コアの切り替え方法 {#switch-exec-core}

まず、あるスレッドを `Low`コアで実行していたとして、`Low`コアがサポートしていない機能/命令が検出された時、例外処理を通知し、`Low`コアでの実行を停止する。  
そして、`High`コアに切り替えると同時に、`High`コアと`Low`コアで共有するキャッシュメモリスレッドの状態をセーブ。  
`High`コアは共有キャッシュメモリにあるスレッドの状態を読み取り、復元、スレッドの実行を開始する。  

コアを切り替え、共有キャッシュメモリにスレッドの状態をセーブする際、他のデータやレジスタの内容を転送することができ、切り替え前のコアが持つキャッシュメモリの内容を一部、または全て転送することも可能となっている。  
これにより、`High`コアでの実行を開始するまでの時間を短くすることができる。  
最初 `Low`コアで実行する時、サポートしていない機能/命令が検出される前に、スレッドの状態を `Low`コア、`High`コア両方のプライベートキャッシュ(L1cache) にセーブし、対応した `High`コアにデータを転送することもできる。  

これらの流れ、操作は異なる順序、並行に実行することも許容されている。  

また、`low power mode` が存在し、これは C-State のような ACPI(Advanced Configuration and Power Interface) 機能に含むことも可能である。  
恐らく、電力管理機能と合わせて、性能が必要な際に `Low`コアから`High`コアに切り替えたり、逆にそこまで性能を必要としない際、`High`コアから`Low`コアに切り替えるのだろう。  
`low power mode` 時ではまず `Low`コアで実行されるが、最初から `High`コアで実行するモードも存在すると思われる。  

SMT機能に関しては、スレッドの状態をセーブするのに必要なキャッシュメモリ量が増えるが、可能である *かもしれない* 。  

### 利点と欠点 {#merit-demerit}
この構成の利点と欠点だが、Intel [Lakefield](/tags/lakefield)と比較して見ていく。  

利点としては、まず[Lakefield](/tags/lakefield) のように、AVX命令のような片方のコア(*Core /SunnyCove*) が対応していて、もう片方(*Atom /Tremont*) は対応していないような命令を、プロセッサ全体では非対応とする必要がないことがあげられる。  
`High`コアのみが対応し、`Low`コアが対応していない機能/命令も実行コアの切り替えによって実行が可能となっている。  
スレッドを実行するコアの切り替えも、[Lakefield](/tags/lakefield) は L3cache を介して行なわれるが、AMD の構成ではそれよりも低階層な L2cache を介すため、低レイテンシ、省電力で行なえるはずだ。  


欠点としては、実行スレッドを切り替える以上、[Lakefield](/tags/lakefield)のように *Atom /Tremont* コアでバックグラウンドタスクを実行する、といったことは不可能であるように思う。  
その場合、8\* `High`コア + 8\* `Low`コアという構成であるとき、見かけ上のコア数は 8-Core となる。  
ここはハイブリッドコア技術における、Intel と AMD のわかりやすい違いと言えるのではないだろうか。  
また、`Low`コアが対応していない機能/命令も `High`コアに切り替えて実行可能だが、その分対応命令は対称的な [Lakefield](/tags/lakefield)よりも切り替え数は増えることが考えられる。  
これを L2cache を介すことでどれだけ補えるか。  

## 余談 {#aside}

近い技術として Arm の `big.LITTLE` が持ち出されたりするが、それでは `big` コアと `LITTLE` コアが同じ命令セットに対応していたのに対し、AMD の実装では高性能コアと省電力コアとで対応する命令セットが異なり、省電力コアの方が機能としては少なくなる。  
厳密に言えばその点で Arm `big.LITTLE` とは異なり、  
非対称コアが共通して対応している命令セットのみを抜き出し、そうでない命令は非対応とした、命令セットでは対称な Intel [Lakefield](/tags/lakefield) の方がずっと近いと言える。  

2種類のコアによって、広い範囲の消費電力比性能をカバーするという点では同じようには思うが。  

個人的な推測を書けば、`Low`コアは `High`コアの存在を前提にガッツリ機能を削っているようにも見える。  
これを実装したプロセッサが実際に世に出るとして、アーキテクチャ名やブランドも `Low`コアと `High`コアそれぞれに付けるのではなく、まとめたものとして付けられるかもしれない。  
