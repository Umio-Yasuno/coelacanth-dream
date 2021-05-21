---
title: "Rust の勉強と CPUID の実践と"
date: 2021-05-21T22:41:49+09:00
draft: false
# tags: [ "", ]
# keywords: [ "", ]
categories: [ "Diary", "Software", "CPU" ]
noindex: false
# summary: ""
---

最近 Rust の勉強も兼ねて CPUID を出力、解析するツールを自作してる。  
今月記事を対して投稿してないのは忙しいとか話が思い付かなかったこともあるが、こうした勉強とツール自作をしていたのも理由の一つ。  
知識の定着のため、ブログに記事を書くこともいいがツールを作って実際に動かすことはもっといい。  
まだ作り始めたばかりだが、ここまでの感想を日記に書くことにした。  

 * [Umio-Yasuno/rust-cpuid-asm](https://github.com/Umio-Yasuno/rust-cpuid-asm)

下は手元の実行例をそのまま貼ったものだが、最近 `<pre></pre>` に `white-space: pre` を CSS に設定していると Firefox 限定でページ幅が想定よりも広がることに気付いたため、`white-space: pre-wrap` を設定している。そのためスマホだと改行されて見にくいかもしれない。  

 > 		CPUID Dump
 > 		========================================================================
 > 		 00000000h_x0: eax=0000000Dh ebx=68747541h ecx=444D4163h edx=69746E65h [AuthenticAMD]
 > 		 00000001h_x0: eax=00800F82h ebx=020C0800h ecx=7ED8320Bh edx=178BFBFFh [F: 17h, M: 8h, S: 2]
 > 		                                                                       [APIC ID: 2]
 > 		                                                                       [Total 12 thread]
 > 		                                                                       [CLFlush: 64B]
 > 		                                                                       [FPU] [MMX] [FXSR]
 > 		                                                                       [HTT]
 > 		                                                                       [SSE/2/3/4.1/4.2]
 > 		                                                                       [FMA]
 > 		                                                                       [POPCNT] [AES] [XSAVE]
 > 		                                                                       [OSXSAVE] [AVX] [F16C]
 > 		                                                                       [RDRAND]
 > 		 00000005h_x0: eax=00000040h ebx=00000040h ecx=00000003h edx=00000011h
 > 		 00000006h_x0: eax=00000004h ebx=00000000h ecx=00000001h edx=00000000h
 > 		 00000007h_x0: eax=00000000h ebx=209C01A9h ecx=00000000h edx=00000000h [FSGSBASE] [BMI1/2] [AVX2]
 > 		                                                                       [SMEP] [RDSEED] [SMAP]
 > 		                                                                       [CLFLUSHOPT]
 > 		                                                                       [SHA]
 > 		 00000007h_x1: eax=00000000h ebx=00000000h ecx=00000000h edx=00000000h
 > 		 0000000Bh_x0: eax=00000000h ebx=00000000h ecx=00000000h edx=00000000h
 > 		 0000000Bh_x1: eax=00000000h ebx=00000000h ecx=00000000h edx=00000000h
 > 		 0000000Bh_x2: eax=00000000h ebx=00000000h ecx=00000000h edx=00000000h
 > 		 0000000Bh_x3: eax=00000000h ebx=00000000h ecx=00000000h edx=00000000h
 > 		 0000000Dh_x0: eax=00000007h ebx=00000340h ecx=00000340h edx=00000000h [X87 SSE AVX]
 > 		 0000000Dh_x1: eax=0000000Fh ebx=00000340h ecx=00000000h edx=00000000h
 > 		 0000000Dh_x2: eax=00000100h ebx=00000240h ecx=00000000h edx=00000000h [XSTATE: size(256)]
 > 		 0000000Dh_x9: eax=00000000h ebx=00000000h ecx=00000000h edx=00000000h
 > 		 0000000Dh_xB: eax=00000000h ebx=00000000h ecx=00000000h edx=00000000h
 > 		 0000000Dh_xC: eax=00000000h ebx=00000000h ecx=00000000h edx=00000000h
 > 		 0000000Fh_x0: eax=00000000h ebx=00000000h ecx=00000000h edx=00000000h
 > 		 00000010h_x0: eax=00000000h ebx=00000000h ecx=00000000h edx=00000000h
 > 		
 > 		 80000000h_x0: eax=8000001Fh ebx=68747541h ecx=444D4163h edx=69746E65h
 > 		 80000001h_x0: eax=00800F82h ebx=20000000h ecx=35C233FFh edx=2FD3FBFFh [PkgType: 2]
 > 		                                                                       [LAHF/SAHF] [LZCNT] [SSE4A]
 > 		                                                                       [3DNow!Prefetch]
 > 		 80000002h_x0: eax=20444D41h ebx=657A7952h ecx=2035206Eh edx=30303632h [AMD Ryzen 5 2600]
 > 		 80000003h_x0: eax=78695320h ebx=726F432Dh ecx=72502065h edx=7365636Fh [ Six-Core Proces]
 > 		 80000004h_x0: eax=20726F73h ebx=20202020h ecx=20202020h edx=00202020h [sor            ]
 > 		 80000005h_x0: eax=FF40FF40h ebx=FF40FF40h ecx=20080140h edx=40040140h [L1D 32K/L1I 64K]
 > 		                                                                       [L1TLB: 64 entry]
 > 		 80000006h_x0: eax=26006400h ebx=66006400h ecx=02006140h edx=00808140h [L2 512K/L3 16M]
 > 		                                                                       [L2dTLB: 4K 1536, 2M 1536
 > 		                                                                                4M  768]
 > 		                                                                       [L2iTLB: 4K 1024, 2M 1024
 > 		                                                                                4M  512]
 > 		 80000007h_x0: eax=00000000h ebx=0000001Bh ecx=00000000h edx=00006599h [RAPL]
 > 		 80000008h_x0: eax=00003030h ebx=00001007h ecx=0000400Bh edx=00000000h [IBPB]
 > 		 80000009h_x0: eax=00000000h ebx=00000000h ecx=00000000h edx=00000000h
 > 		 8000000Ah_x0: eax=00000001h ebx=00008000h ecx=00000000h edx=0001BCFFh
 > 		 80000019h_x0: eax=F040F040h ebx=00000000h ecx=00000000h edx=00000000h [L2TLB 1G: D 0, I 0]
 > 		 8000001Ah_x0: eax=00000003h ebx=00000000h ecx=00000000h edx=00000000h [FP128 MOVU]
 > 		 8000001Bh_x0: eax=000003FFh ebx=00000000h ecx=00000000h edx=00000000h
 > 		 8000001Ch_x0: eax=00000000h ebx=00000000h ecx=00000000h edx=00000000h
 > 		 8000001Dh_x0: eax=00004121h ebx=01C0003Fh ecx=0000003Fh edx=00000000h
 > 		 8000001Eh_x0: eax=00000002h ebx=00000101h ecx=00000000h edx=00000000h [Core ID: 1]
 > 		                                                                       [2 thread per core]
 > 		 8000001Fh_x0: eax=0000000Fh ebx=0000016Fh ecx=0000000Fh edx=00000000h [SME SEV(-ES)]
 > 		 80000020h_x0: eax=00000000h ebx=00000000h ecx=00000000h edx=00000000h
 > 		 80000021h_x0: eax=00000000h ebx=00000000h ecx=00000000h edx=00000000h

CPUID周りのドキュメントを読んでいて面白いと思ったのは、AMD CPU だと性能最適化に役立つ情報として FPUパイプラインのデータ幅が CPUID から知ることができるようになっていることだ。  
具体的には EAXレジスタに `0x8000_001A` をセットして CPUID命令を実行すると EAXレジスタに値が出力され、その 1-bit目が 1 であれば 128-bit幅 (FP128)、3-bit目が 1 であれば 256-bit幅 (FP256) の FPUパイプラインがその AMD CPU に実装されていることがわかる。  
上の **AMD Ryzen 5 2600** の例では FP128 になっているし、 *Zen 2/3 アーキテクチャ* のプロセッサで実行すれば FP256 となるだろう。  
その値はまだほとんど使われておらず予約領域となっており、今後 AMD CPU で機能の追加、FPUパイプラインのデータ幅拡張が行われれば情報が追加されると思われる。  

あと CPUID から対応命令を知ることができるようになっているが、Intel も AMD も自分の所の CPU に実装している命令についてしかドキュメントに書かないものだから、網羅しようとすると何回も両者のドキュメントを往復することになる。  
英語版 Wikipedia の CPUID の記事は、最近追加された命令は含まれていないものの、命令とその対応を示すビットフラグの位置がある程度統合されているので助けられた。[^wiki-cpuid]  

[^wiki-cpuid]: [CPUID - Wikipedia](https://en.wikipedia.org/wiki/CPUID)

自作ツール `cpuid_dump` では Rust の `asm!` 機能を使ってアセンブリで記述した CPUID命令を実行している。そして `asm!` はまだ unstable な機能なため、Nightly版の Rust でないとコンパイルできないので注意。  
その `asm!` だが、かなり理解しやすいと感じた。  
CPUID命令は EAXレジスタ、時には ECXレジスタにもまず特定の値をセットしてから実行すると、情報がEAX/EBX/ECX/EDXレジスタに出力される。EAXレジスタにセットする値は leaf、ECXレジスタにセットする値は subleaf とも呼ばれる。[^rs-asm]  

[^rs-asm]: [asm - The Rust Unstable Book](https://doc.rust-lang.org/beta/unstable-book/library-features/asm.html)

インラインアセンブリは C言語 (GCC) だと以下のように書くが、アセンブリ完全初心者の自分には難しい。正直入力元出力先の記法をまだ理解できていない。少し試してそれからは Rust に移った。  

```
    __asm__ volatile (
        "cpuid\n"
            : "=a" (a[0]), "=b" (a[1]), "=c" (a[2]), "=d" (a[3])
            : "0" (i)
    );
```

Rust では以下のようにインラインアセンブリ書ける。EAXレジスタに値をセット、後に EAXレジスタに値が出力されることがパッと見てすぐに理解できる。  
レジスタ名も記述する必要があるが、ドキュメントで使われている名称をそのまま使えるためむしろ理解の助けになる。完全初心者の自分には Rust の方がずっと楽だったのは言うまでもない。  

結局さらに楽をしようとしてマクロを使うようになり、わかりやすさという利点は早々に失われたが。それでも入口としてとてもよく機能してくれたため、それはそれでいいのかもしれない。  

```
    unsafe {
        asm!{
            "cpuid",
            inlateout("eax") 0 => a[0],
            lateout("ebx") a[1],
            lateout("ecx") a[2],
            lateout("edx") a[3],
        }
    }
```

それとコア間レイテンシを測定するツールも作ったがこれは、Github に [rigtorp/c2clat](https://github.com/rigtorp/c2clat) という C++ で 140行程で書かれたシンプルで素晴らしいプログラムのソースコードが MITライセンスで公開されていたため、それを Rust にポーティングしたもの。  
コア間レイテンシは Ryzenプロセッサではよく注目される要素で、個人的にも興味があった。  
自分用として結果を Markdown形式で出力するオプションや、gnuplot 用のオプションをわずかに修正したりもしたが、自分が加えたのはそれだけだ。  
知識の実践というよりは、Rust におけるスレッド処理の書き方を学ぶことができた。  
ただ Rust版には少し問題があり、オリジナルでは `alignas (64)` をアトミック変数の前に書くことでキャッシュラインに収まるようにしている。Rust版でもそれと近いことをしようとしたがうまくいかず、結果オリジナル版よりも若干大きいレイテンシが測定される。  

今後そういった部分の修正もやっていきたいが、色んな CPU のマイクロベンチマークを Rust で実装することも目標にしていきたい。  
