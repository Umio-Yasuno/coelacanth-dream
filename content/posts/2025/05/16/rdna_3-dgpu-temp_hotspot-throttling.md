---
title: "RDNA 3 dGPU で TEMP_HOTSPOT スロットリングがほぼ常時検出される理由"
date: 2025-05-16T10:20:10+09:00
draft: false
categories: [ "Hardware", "AMD", "GPU" ]
tags: [ "Linux_Kernel", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

## TL;DR {#tldr}
AMDGPU ドライバーでは SMU (System Management Unit) が報告する `ThrottlingPercentage` が 0 でなければスロットリングのフラグビットを有効にするが、RDNA 3 dGPU では `TEMP_HOTSPOT` の `ThrottlingPercentage` がほぼ常時 1 以上だと報告されるため。  

## gpu_metrics と indep_throttle_status {#sysfs}
AMDGPU ドライバーと最近の AMD APU/GPU の組み合わせでは `gpu_metrics` sysfs ファイルをサポートしており、これには GPU の各種動作クロックや電圧、消費電力、温度、スロットリングステータスといった情報がまとめられている。  
他の sysfs ファイルから取得できる情報も含まれているが、`gpu_metrics` は単一の sysfs ファイルにまとめられているため、タイミングがずれることなくまとめて取得することができる。  

`gpu_metrics` はテキスト形式ではなくバイナリ形式であるため、ユーザースペースのアプリケーションは構造体の定義と、ヘッダーから取得できるバージョン情報を基にして最終的な構造体 (`gpu_metrics_v{}_{}`) に変換する必要がある。  
構造体の定義は Linux Kernel の `drivers/gpu/drm/amd/include/kgd_pp_interface.h` に公開されている。  

 * <https://www.kernel.org/doc/html/latest/gpu/amdgpu/thermal.html#gpu-metrics>

そして一部の `gpu_metrics_v{}_{}` 構造体には `throttle_status` と `indep_throttle_status` フィールドが存在している。  
`throttle_status` は SMU が報告するスロットリング情報をフラグビットセットの形式に変換したものであり、`indep_throttle_status` はそれを共通のフラグビットセットに変換したものとなる。  
そのため、アプリケーションは `indep_throttle_status` とその各スロットリングのフラグビットの位置をサポートしていれば、APU/GPU の種類に依存したコードを書くことなくスロットリング情報を取得することができる。  
`indep_throttle_status` からのスロットリング情報の取得をサポートしているアプリケーションは、自分が知っている限り [MangoHud](https://github.com/flightlessmango/MangoHud) と、
[libdrm-amdgpu-sys-rs](https://github.com/Umio-Yasuno/libdrm-amdgpu-sys-rs) を採用している [amdgpu_top](https://github.com/Umio-Yasuno/amdgpu_top) と [LACT](https://github.com/ilya-zlobintsev/LACT) だけである。  
AMD が公式に開発して公開しているツールには、`gpu_metrics` のデコードまではサポートしていても、スロットリング情報の検出までサポートしているものは無い。  

## TEMP_HOTSPOT {#temp_hotspot}
スロットリング情報の取得はほとんどの GPU でうまく機能するが、RDNA 3 dGPU だけは何故かほぼ常時 `TEMP_HOTSPOT` が検出される問題が発生していた。  

 * [`throttle_status` in `gpu_metrics` for Navi31 always show TEMP_HOTSPOT (#3251) · Issue · drm/amd](https://gitlab.freedesktop.org/drm/amd/-/issues/3251)
 * [Thinks that I'm constantly thermal throttling / power limited · Issue #1243 · flightlessmango/MangoHud](https://github.com/flightlessmango/MangoHud/issues/1243)

一部の RDNA 2 dGPU、*Navi22/Navy Flounder* と *Navi23/Dimgrey Cavefish* にもスロットリング情報がアイドル状態でも検出される問題が発生していたが、これは AMDGPU ドライバー側の処理に抜けていた部分があったためである。[^rdna_2]  
しかし、RDNA 3 dGPU 関係のコードを読んでもドライバー側に原因があるようには思えなかった。  
そのため自分は最初ファームウェア側の問題と考え、ドライバー開発者からの返答を待つことにした。  
だが、特に進展が無いまま最初に報告されてから 1年が経過した。  
この問題はアイドル時や GPU 温度が低い状態でも発生しており、明らかに異常な挙動ではあったが、性能や動作クロックからして実際にはスロットリングは発生していないと考えられたからだろう。  
基本、ファームウェアやドライバーに対する問題は、性能や安定性に影響が無ければ対応の優先度は低くなる。  
自分が知っているものでは、Navi33 で GFXOFF 機能が有効化されているとアイドル時の GPU 使用率が瞬間的に 100% になる、という問題があるが、これも報告されてから執筆時点で 9ヶ月経ってるが、特に修正される様子はない。[^rx7600-gfxoff]  

[^rdna_2]: [【雑記】 RX 6600 の購入とバグ報告と更新頻度と | Coelacanth's Dream](/posts/2023/08/03/coelacanth-diary-2023-08-03/)
[^rx7600-gfxoff]: [AMD GPU usage peaks with system in idle - Radeon RX 7600 XT (#3549) · Issue · drm/amd](https://gitlab.freedesktop.org/drm/amd/-/issues/3549)

途中でドライバー側で `TEMP_HOTSPOT` スロットリングのフラグビットを有効化しないようにすることで、暫定的ではあるが対策することを提案したが、ドライバー開発者からの反応は無かった。  
また、自分で RX 7600 を購入して調査、検証することを検討もしたが、実行には踏み切れなかった。調べた結果、仮にファームウェアの問題と判明してもそれ以上やれることはなく、また問題が放置される可能性が充分にあった。  
それに 4-5万する PCパーツを気軽に買えるほど自分は金持ちじゃなかった。ずっと前はインターネット上の活動で得た収益を検証用パーツの購入費にあてる構想があったが、今は完全に諦めている。  

修正されないまま RDNA 4 dGPU、RX 9070 シリーズが発表され、同じ問題が発生することを懸念していたが、発売されてからしばらく経っても報告されることはなかった。  
コードを読んでも RDNA 3 dGPU とスロットリング情報に関してはほぼ同じである。  
RDNA 3 dGPU でだけ発生する点が気になり、もう一度調査してみることにした。  

まず、ファームウェア、ハードウェア側である SMU が更新するデータ、`SmuMetrics_t` 内に `ThrottlingPercentage` があり、これは `uint8_t` の配列となっている。  
`ThrottlingPercentage` の各要素にはそれぞれ対応したスロットリング情報の度合い (%) が収められている。  
AMDGPU ドライバーの処理としては、その度合いが非 0 であればスロットリングのフラグビットを有効化するような実装になっていた。これは RDNA 2 dGPU、RDNA 4 dGPU でも同様となっている。  
自分が手持ちの RX 6600 (Navi23/Dimgrey Cavefish) で検証した結果では、それぞれのスロットリング情報に閾値と限界値が設定されており、その 2つの値の間で度合いが 1-100% で表現され、限界値に近いほど度合いが大きくなるようになっていた。  
次に RDNA 3 dGPU の `ThrottlingPercentage` が実際にどうなっているか確認する必要があったが、そこでまた行き詰まった。  
AMDGPU ドライバー内部で使用しているデータの確認方法を、Linux Kernel のログに出力する方法しか知らなかったからだ。  
この方法の場合、自分では検証できない環境に対するパッチを書き、それを他者が適用、ビルドし、起動する必要がある。不可能ではないが、労力と時間を考えると極力取りたくない方法だ。  
だがその後、他の問題に対して [bpftrace](https://github.com/bpftrace/bpftrace/) を使用して AMDGPU ドライバー内部のデータを確認しているユーザーを見付け、`bpftrace` の存在を知った。[^bpftrace]  
`bpftrace` の技術的な詳細は省くが、単一のスクリプトファイルで Linux Kernel の動的なトレースが可能になる、かなり便利なツールである。  
自分の環境では完全な検証ができないことに変わりはないが、修正と実行においては先の方法とは比べ物にならないほど簡単だ。  

[^bpftrace]: <https://gitlab.freedesktop.org/drm/amd/-/issues/2321#note_1746183>

RX 6600 で検証し、ある程度 `bpftrace` の使い方を学んだ後、*Navi31, Navi32* 向けの検証用スクリプトを書いて公開した。*Navi33* が対象に含まれていないのは使用している構造体が異なるため。  
そして他者がスクリプトを実行した結果を確認したところ、確かにアイドル状態でも `ThrottlePercentage` は非 0 ではあるが値は 1-30 (%) で安定し、ストレステスト実行時は 80-100 (%) となっていた。  
値がランダムではなく、温度や負荷に合わせて `ThrottlingPercentage` も変化することから、機能自体は正常だが、閾値が異常に低い設定になっているという推測が立てられる。  
そこで自分は `TEMP_HOTSPOT` に限り `ThrottlingPercentage` が 80 以上の場合にフラグビットを有効化するパッチを書いて提案した。  
単にフラグビットの有効化にあたってドライバー側に閾値を設定するだけであり、ハードウェアの動作には影響しない。根本的な修正にはならないが、ユーザーから見れば正常に機能するようになるはずだ。  
また、ファームウェアでの修正が期待できないのであれば、ドライバーで対処するのが修正方法として現実的だと考えた。  
しかし、ドライバー開発者からのパッチに対する反応は良くなかった。  
ドライバー開発者によれば、`gpu_metrics` は SMU、ファームウェア (PMFW) からの情報を "出来る限り" 生のまま出力するように設計されているとのことだった。つまり、ドライバー側で閾値を設定するのはそれに反しているということなのだろう。  
正直に言えば、ファームウェアが 0-100 (%) で表現しているものをフラグビットに変換している時点で今更であるように思えた。  

気付けば問題に対してどのレベルで対処し、責任を持つべきなのか、という別の複雑な問題になっていた。  
他 GPU と異なる挙動をするファームウェアが悪いのか？ その問題を把握せず、対処もせずにユーザースペースに公開するドライバーが悪いのか？ ドライバーからの情報を信頼しきって、そのままユーザーに対して表示するアプリケーションが悪いのか？  

ユーザースペース側で回避策を実装して対処すること自体は可能だし、実装が難しい訳でもなかった。  
しかしそうなると、ライブラリあるいはアプリケーションごとに回避策を実装する必要があり、将来 `gpu_metrics` とその `indeo_throttle_status` をサポートするものが出てきた時も回避策の実装が必要になってくる。  
また、回避策を実装する必要があるということは、ファームウェアとドライバーからの情報が信頼できないということを意味する。  
だが、結局ユーザースペースに回避策を実装することにした。  
ファームウェアが修正されるかも不明で、AMDGPU ドライバーへのパッチ受け入れも期待できない以上、そうするしかない。  
自分が開発している `libdrm-amdgpu-sys-rs` にはホットスポット温度 (ジャンクション温度) が 90度未満なのに有効化されている `TEMP_HOTSPOT` フラグビットを信用せず、クリアすることにした。  
温度による閾値の設定でもある程度実用的に機能する、ようには見せられるはずだ。  
`MangoHud` に対しても自分で回避策を実装するプルリクエストを作成した。[^mangohud-pr]
問題について把握している自分が対応した方が早いだろうと考えたためだが、これまで関わってきたことによる責任感もあった。  
最初は `libdrm-amdgpu-sys-rs` と同様の温度による閾値の設定を提案したが、`Mangohud` の開発者である flightlessmango 氏との話し合いで常に `TEMP_HOTSPOT` フラグビットをクリアすることになった。  
無効なスロットリング情報が報告されるのであれば、部分的ではあるが最初から信用するべきではない、ということだろう。  

[^mangohud-pr]: [amdgpu: implement the invalid TEMP_HOTSPOT throttling workaround for RDNA 3 dGPU & some fixes by Umio-Yasuno · Pull Request #1690 · flightlessmango/MangoHud](https://github.com/flightlessmango/MangoHud/pull/1690)

今にして思うと、もっと早くに回避策を実装していれば混乱するユーザーをもっと減らせただろう。だが、問題の原因が不明で、ドライバー開発者からの説明も無かったため、そう判断するのは難しかった。  

今後修正されたファームウェアがリリースされるかは不明、そもそも内部的にこの問題が共有されているかも不明である。  
結果としては、ファームウェアの問題とされるものに対して、1年越しにユーザースペースで回避策を実装するという若干苦いものとなったが、ユーザーからの問題報告に対して優先度が存在することは理解しているため、あまり深くは考えていない。  
今回で `bpftrace` の使い方を学べたため、次はもっと効率良く原因を突き止められるはずだ。  
今後もファームウェア側の問題に対して。ユーザースペースで回避策を実装する必要があるのかと言えば、それは結局の所、問題によるだろう。  
何しろ `gpu_metrics` も `indep_throttle_status` も、ユーザースペースに公開されてはいるが、使い方に関しては全くドキュメント化されていない。それなのに問題を報告されても、あまり解決に向けて積極的にはなれないのかもしれない。  
とりあえず、自分は今後も出来る範囲で AMDGPU ドライバーやそのコミュニティに協力していくつもりである。  
