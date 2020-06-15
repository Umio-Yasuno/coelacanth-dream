---
title: "SHA-NI命令セットへの対応で最大1.5倍 Git の高速化が可能"
date: 2020-04-10T10:56:35+09:00
draft: false
tags: [ "Zen", "Zen2", ]
keywords: [ "", ]
categories: [ "Software", ]
noindex: false
---

[KeyDB](https://keydb.dev/) の開発者である John Sully氏が、有名なバージョン管理ソフト [Git](https://git-scm.com) 内の一部を SHA-NI命令セットに対応させることで、Ryzen CPUにおける実行が最大1.5倍高速されたとする記事を投稿した。  

[Optimizing Git For Ryzen CPUs (1.5x Faster) ·](https://docs.keydb.dev/blog/2020/04/08/blog-post/)

## SHA-NI命令セット
2017年に登場し、ハードウェア業界に新たな風を吹き込んだ *Zenアーキテクチャ* の特徴の1つとして、SHA-NI命令セットをサポートしているというものがある。  
*SHA* は *Secure Hash Algorithm* 、*NI* は *New Instruction* の略であり、その名の通り暗号化を高速に行なうために用いられる。  

現行で対応しているCPUは、*Hygon* を除く[^1]、*AMD Zen* 、*AMD Zen 2* アーキテクチャベースのCPU、(Athlon, Ryzen, Threadripper, EPYC)  
*Intel Sunny Cove* アーキテクチャベースの *Ice Lake (Client)* だけとなる。  

{{< ins datetime="2020-06-15T11:27:19" >}}

LLVMのソースコードを見るに、Intel Atom が *Goldmont* より対応しているようです。  
{{< link >}}<https://github.com/llvm/llvm-project/blob/d5c28c4094324e94f6eee403022ca21c8d76998e/clang/lib/Basic/Targets/X86.cpp#L260>{{< /link >}}
お詫びするとともに、ここに追記し、修正致します。  

{{< /ins >}}

製品が未登場ではあるが対応していることが確認されているCPUは、*Ice Lake (Server)* 、*Tiger Lake* 。  
キャンセルされてしまった *Cannon Lake* も対応していた。[^2]  

このように対応しているCPUはまだ少なく、シェアを大きく占める Intel CPU では、対応したサーバ向け、デスクトップ向けCPUが出ていないということもあり、ソフトウェア側の対応はまだ進んでいないとされている。  

[^1]: [Our Hygon Systems: 8-core Dhyana and dual 32-core Dhyana Plus - Testing a Chinese x86 CPU: A Deep Dive into Zen-based Hygon Dhyana Processors](https://www.anandtech.com/show/15493/hygon-dhyana-reviewed-chinese-x86-cpus-amd/2)
[^2]: [x86 Options (Using the GNU Compiler Collection (GCC))](https://gcc.gnu.org/onlinedocs/gcc/x86-Options.html)

## 対応により最大1.5倍の高速化
そんな中、John Sully氏はソフトウェアを SHA-NI命令セットに対応させることで、Ryzen CPUはより優れた性能を発揮できるのではないかと考え、Git への実装に至った。  

Git はバージョン管理にSHA-1によりハッシュ値を用いるため[^3]、理論上ではSHA-NI命令セットへの対応、最適化によって一部操作の高速化が期待できる。  
結果、Linuxのレポジトリにおいては、1GBのファイルを追加する `git add` コマンドで 1.5倍、  
ローカルのブランチ、コミットを切り替える `git checkout` コマンドで 1.06倍、内部データベースのエラー、整合性をチェックする `git fsck` コマンドで 1.4倍の高速化が確認された。  
巨大なファイルを取り扱う処理で効果が顕著に見られるようだ。  
一方で、新たなブランチを追加する `git merge`、単純にGitリポジトリを複製する `git clone` コマンドでは、SHA-NI命令セット対応前の結果とほぼ変わらないものとなった。  

John Sully氏はディスクI/Oがボトルネックとなっていなければ、他のコマンドでも性能の改善が見込めると考えている。  
実際、先の `git clone` の結果もLinux Kernelのリポジトリは巨大であるため、ディスクI/Oがボトルネックになっていたものと思われる。  

[^3]: [Git - リビジョンの選択](https://git-scm.com/book/ja/v2/Git-%E3%81%AE%E3%81%95%E3%81%BE%E3%81%96%E3%81%BE%E3%81%AA%E3%83%84%E3%83%BC%E3%83%AB-%E3%83%AA%E3%83%93%E3%82%B8%E3%83%A7%E3%83%B3%E3%81%AE%E9%81%B8%E6%8A%9E)

新しい機能、命令への対応により、一部ユーザは確かにその恩恵を得られるが、やはりその裏でほとんど得られないユーザも存在する。そこでは得られる(シェア)に対するコスト効率という考えが働くが、John Sully氏は Git のような大規模なソフトウェアに数時間で高速化を成し遂げた。  
氏はAMD EPYCがデータセンタに広く採用され、それによって Zenアーキテクチャへの最適化が為されたソフトウェアが増えることを期待し、  
そして、氏が変更した Git は <https://github.com/JohnSully/git> に公開しているが、これはデモンストレーション用のプロトタイプであり、本番環境に使用することはまだお勧めしないとして、記事を締めている。  

{{< ref >}}

 * [One-Way Hash Primitives](https://software.intel.com/en-us/ipp-crypto-reference-one-way-hash-primitives)

{{< /ref >}}
