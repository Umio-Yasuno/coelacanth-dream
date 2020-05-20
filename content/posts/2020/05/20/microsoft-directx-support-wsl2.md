---
title: "Microsoft が WSL2 で DirectX をサポートする計画を発表"
date: 2020-05-20T02:44:51+09:00
draft: false
# tags: [ "", ]
keywords: [ "", ]
categories: [ "Software", "GPU" ]
noindex: false
---

Microsoft は開催中の開発者向けイベント、[Microsoft Build 2020](https://news.microsoft.com/build2020/)の中で、Windows向けグラフィクスAPI DirectX を WSL2 (Windows Subsystem for Linux 2) でサポートすることを発表した。  
機械学習 API DirectML や NVIDIA CUDA といった GPUコンピューティングも WSL2 下で実行可能となる。  
また、リモートデスクトップを用いて、Linux向けGUIアプリケーションを WSL2 から動作させる機能の将来的なサポートも発表している。  
{{< link >}}<cite>[DirectX ❤ Linux | DirectX Developer Blog](https://devblogs.microsoft.com/directx/directx-heart-linux/)</cite>{{< /link >}}
{{< link >}}<cite>[The Windows Subsystem for Linux BUILD 2020 Summary | Windows Command Line](https://devblogs.microsoft.com/commandline/the-windows-subsystem-for-linux-build-2020-summary/){{< /link >}}

## Direct X on "WSL"

仕組みとしては、Linux Kernel に `Dxgkrnl` と呼ぶ新たな KMD (Kernel Mode Driver) を実装し、Windows OS をホストOSとする Linux は `Dxgkrnl` を、`Dxgkrnl` は仮想的なバス (VM Bus) を通して Windows OS内のGPUドライバー、そして物理GPUと通信することで、DirectX 12 API を使用する。  
これは WDDM (Windows Display Driver Model) v2.9 の仕様に統合される。  

それと同時に DirectX のライブラリが Linux向けにリリースされる。  
DirectX 12 の Linux向けライブラリ、`libd3d12.so` は Windows向けである `d3d12.dll` と同じソースコードからコンパイルされており、仮想化によるオーバーヘッドを除き、同等の機能と性能を提供するとしている。  
現時点で Linux向けの DirextX 12 APIはオフスクリーンレンダリングとコンピュート機能を実行できるが、画面に直接レンダリングするためのスワップチェイン機能はサポートしていない。  
ただこれは、その後に笑顔の顔文字付きで "yet" と書いており、将来的にサポートするつもりだと思われる。  
UMD (User Mode Driver) と KMD が通信する際に用いられるレイヤー、DXGI の Linux向けライブラリ `libdxcore.so` もリリースされる。  
そして、WSL2 側の UMD は GPUメーカーと連携し、WSL環境で実行可能となるよう再コンパイルされるとのこと。  
これもまた、WDDM v2.9 ドライバーに統合される。  

留意しておきたいのは、`libd3d12.so` 、`libdxcore.so` はクローズドソースであり、事前にコンパイルされたバイナリのみが、Windows の一部として出荷される。  
GPUメーカーが提供する DirectX 12 UMD も当然クローズドソースだ。  
そのため、WSL2以外 (WindowsをホストOSとしない Linux OS) は UMD が存在しないため、動作させることはできない。  

また、Linux 環境下で DirectX を実行するソフトウェアでは、[Wine](https://www.winehq.org/#carouselScreenshots) / [Proton](https://github.com/ValveSoftware/Proton/)が有名だが、Wine は API 呼び出しを変換する形で実行しているため、今回の Microsoft が行なう Linux へのさらなる歩み寄りを、それらに活かせるかは微妙なところと思われる。  

{{< ref >}}

 * [DirectX ❤ Linux | DirectX Developer Blog](https://devblogs.microsoft.com/directx/directx-heart-linux/)
 * <cite>[Microsoft Announces Direct3D 12 For Linux / WSL2 - Phoronix](https://www.phoronix.com/scan.php?page=news_item&px=Microsoft-DX12-WSL2)</cite>

{{< /ref >}}
