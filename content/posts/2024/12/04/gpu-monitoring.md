---
title: "GPU を起こさないモニタリング"
date: 2024-12-04T10:04:00+09:00
draft: false
categories: [ "Diary", "Note", "GPU" ]
tags: [ "Rust", ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

自作の AMD GPU 向けモニタリングツール `amdgpu_top` で、v0.9.0 で dGPU が D3 ステートに移行可能に、v0.9.2 で D3 の dGPU を起こさずに (D0 にせずに) ツール側の起動と初期化を可能にする作業をした。その備忘録。  
ここでの解説は個人的な限られた検証によるものの含まれる。  

## PCI デバイスのステート {#state}
PCI デバイスには複数の電源状態 (D0, D1, D2, D3hot/D3cold) があり、かなりざっくり言えば D0 が起動状態 (フル)、D3hot/D3cold は待機/停止状態とされている。  

 * [PCI Power Management — The Linux Kernel documentation](https://docs.kernel.org/power/pci.html)
 * [デバイスの電源状態 - Windows drivers | Microsoft Learn](https://learn.microsoft.com/ja-jp/windows-hardware/drivers/kernel/device-power-states)
 * [【短期集中連載】大原雄介の最新インターフェイス動向](https://pc.watch.impress.co.jp/docs/2009/0318/interface04.htm)

一般的なデスクトップ PC の構成で見られる dGPU 側の映像出力を使用している場合では dGPU が D3hot/D3cold に移行することはないが、  
APU/iGPU と dGPU を組み合わせたノート PC でよくある構成で、APU/iGPU 側の映像出力を使用しており、かつ dGPU を使用するアプリケーションが起動していない場合は dGPU が D3hot/D3cold に移行可能となる。  

D3cold/D3hot ステートの dGPU は様々な理由で起動 (D0 への移行) する。  
一般のケースではゲームやアプリケーションからの使用によって起動する。ただ厳密にはデバイスハンドルの初期化に必要な fd (file descriptor) を取得しただけでも起動するし、一部 sysfs の読み取りや hwmon 経由のセンサー値の読み取りだけでも起動する。  
これはモニタリングツール側からするとかなり厄介で、何の回避策も実装していないとモニタリングツール自体が dGPU を起動、そして D3hot/D3cold への移行を妨害し、消費電力を増やす最大の要因となってしまう。  
これはノート PC では特に大きな問題となる。  

いくつもの GPU モニタリングツールでこの問題が報告されているが、複数のベンダーの GPU のサポート同時に、D3hot/D3cold ステートのデバイスも対象に含めると複雑度が増すためか、対応が中々進まないのが現状となる。  
その中で `amdgpu_top` は名の通り AMD GPU のみを対象にしたモニタリングツールであるため (最近 XDNA NPU のサポートを追加したが)、比較的簡単に対応することができた。  

 * [PRIME Reporting · Issue #90 · Umio-Yasuno/amdgpu_top](https://github.com/Umio-Yasuno/amdgpu_top/issues/90)
 * [nvtop keeps otherwise-idle dGPU awake, significantly increasing system power consumption · Issue #230 · Syllo/nvtop](https://github.com/Syllo/nvtop/issues/230)
   * [GPU stats keep Nvidia/AMD dGPU powered up and draining power (#30) · イシュー · mission-center-devs / Mission Center · GitLab](https://gitlab.com/mission-center-devs/mission-center/-/issues/30)
 * [Nvidia RTD3 Power Management Unawareness Bug in Hybrid Graphics Laptops · Issue #207 · nokyan/resources](https://github.com/nokyan/resources/issues/207)
 * [Polling Nvidia temperature keeps GPU awake · Issue #1291 · ClementTsang/bottom](https://github.com/ClementTsang/bottom/issues/1291)

## dGPU を D3hot/D3cold に移行可能にする {#d3}
まず `amdgpu_top` を起動した状態でも dGPU が D3hot/D3cold に移行可能に、また dGPU がゲーム等によって起動したらモニタリングを再開する作業から始めた。  
自分の PC は CPU が Ryzen 5 5600G APU であるため、検証環境を用意することは簡単だった。  

D3hot/D3cold に移行可能にするには、一部の sysfs とセンサー値の読み取りを停止すればいいが、その基準を決める必要がある。  
単に dGPU 全体の使用率が 0% の時に停止する方法でもいいかもしれないが、ブラウジングのように断続的に GPU 負荷が発生するものだとうまくいかない場合がある。  
また、読み取りを停止したことをユーザーに通知することを考えると、0% になった途端に停止するのはユーザーにとって使いづらいだろう。  
デバイスの電源状態は `<sysfs path>/power/` 下にある `runtime_status` から取得できるが、モニタリングツール自体が D3hot/D3cold への移行を妨害するため、基準には使うことができない。  

そこで `amdgpu_top` では GPU を使用しているプロセス数で判断することとした。  
`amdgpu_top` は GPU プロセスの検出と fdinfo を利用したプロセスごとの GPU 使用率の取得に対応している。  
dGPU を使用するプロセスが 1つだけの場合、つまり `amdgpu_top` だけの場合に sysfs とセンサー値の読み取りを停止するようにした。  
またデバイスハンドルも解放するようにしたが、D3hot/D3cold への移行だけが目的なら解放する必要はないかもしれない。自分は実装において表現を分かりやすくするためにした。  
ユーザーへの通知については、センサー値等は `Option<T>` で表現していたため、`None` の場合の表示を追加するだけでよかった。  

この基準はサーバー用途等の一部のケースではうまく機能しない可能性はあるが、一般的なデスクトップ用途であれば充分なはずだ。  

モニタリングを再開する基準については、dGPU を使用するプロセスが 2つ以上、かつ `<sysfs path>/power/runtime_status` が `active` となっている場合とした。  

## GPU を起こさずにモニタリングする {#wakeup}
D3hot/D3cold ステートの GPU を起こさずにモニタリングツールを起動するには、GPU 情報をまとめた構造体の初期化を後回しにして、かつ D0 ステートになっているか定期的にチェックするデバイスとして登録する必要がある。  

`amdgpu_top` には TUI, SMI, GUI, JSON の複数のモードがあるが、結局それぞれに処理を実装して対応した。  
D3hot/D3cold に移行可能にするだけなら、複数のモードで共通する部分の変更で対応が済んだが、アプリケーションの初期化を後回しにするため、それぞれでの対応が必要だった。  

AMD GPU は DeviceID と RevisionID だけでデバイス名を判別できるため、何のデバイスが D3hot/D3cold ステートかわかりやすくすることができたが、libdrm、OpenGL や Vulkan といった API を経由してデバイス名を取得するアプリケーションでは難しいだろう。  

モニタリングツール自体がハードウェアのリソースや電力を多く消費しないようにする、というのは当たり前の話に思えるかもしれないが、デバイスの電源状態が絡むと複雑になり、実装が難しくなりやすい。  
