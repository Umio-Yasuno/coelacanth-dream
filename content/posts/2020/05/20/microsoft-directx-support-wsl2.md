---
title: "Microsoft が WSL2 の DirectX サポートを発表"
date: 2020-05-20T02:44:51+09:00
draft: false
# tags: [ "", ]
keywords: [ "", ]
categories: [ "Software", "GPU" ]
noindex: false
---

Microsoft は開催中の開発者向けイベント、[Microsoft Build 2020](https://news.microsoft.com/build2020/)の中で、Windows向けグラフィクスAPI DirectX を WSL (Windows Subsystem for Linux 2) でサポートすることを発表した。  
{{< link >}}[DirectX ❤ Linux | DirectX Developer Blog](https://devblogs.microsoft.com/directx/directx-heart-linux/){{< /link >}}

Linux Kernel に `Dxgkrnl` と呼ぶ新たな KMD (Kernel Mode Driver) を実装し、Windows OS をホストOSとする Linux は `Dxgkrnl` を、`Dxgkrnl` は仮想的なバス (VM Bus) を通して Windows OS内のGPUドライバー、そして物理GPUと通信する。  
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

また、Linux 環境下で DirectX を実行する [Wine](https://www.winehq.org/#carouselScreenshots) / [Proton](https://github.com/ValveSoftware/Proton/) は、API 呼び出しを変換する形で実現しているため、今回の Microsoft が行なう Linux へのさらなる歩み寄りをそれらに活かせるかは微妙なところだ。  
