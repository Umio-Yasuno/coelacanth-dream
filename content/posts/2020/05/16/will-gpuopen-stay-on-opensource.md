---
title: "GPUOpen はオープンであり続けるか？"
date: 2020-05-16T16:11:15+09:00
draft: false
# tags: [ "", ]
keywords: [ "", ]
categories: [ "AMD" ]
noindex: false
---

AMD が取り組む、PCゲーマーにゲームのグラフィック向上と快適動作を提供するためのオープンソースプロジェクト、[GPUOpen](https://gpuopen.com/)。  
その成果を発信する公式サイトが公開から4年近く経ち、2020/05/11 にリニューアルされた。  
しかし、それから新たにリリースされた CPU + GPU 環境のための光の交差計算用ライブラリ、[RadeonRays](https://github.com/GPUOpen-LibrariesAndSDKs/RadeonRays_SDK)のバージョン 4.0 は、従来行なってきたオープンソースでの提供ではなく、コンパイル済みのバイナリのみというクローズドソースでの提供するというものだった。  
そのことが一部ユーザー間で話題となり、"炎上"のような状態となった。  

## GPUOpen
GPUOpen は 2016年、*Polarisアーキテクチャ* 採用GPUの発表と同時期に開始され、これまでに、

 * アプリケーションの動作FPSを取得する [OCAT](https://github.com/GPUOpen-Tools/ocat)
 * GPUの動作状況を取得し、最適化の手助けをする各プロファイルツール
    * [Radeon Compute Profiler](https://github.com/GPUOpen-Tools/radeon_compute_profiler)
    * [GPU Performance API](https://github.com/GPUOpen-Tools/gpu_performance_api)
    * [RGA (Radeon GPU Analyzer)](https://github.com/GPUOpen-Tools/radeon_gpu_analyzer)
    * [Radeon™ GPU Profiler](https://github.com/GPUOpen-Tools/radeon_gpu_profiler)
    * [Radeon™ Memory Visualizer](https://github.com/GPUOpen-Tools/radeon_memory_visualizer)
 * AMD GPU のマルチメディア処理機能を使用するためのSDK [Advanced Media Framework (AMF)](https://github.com/GPUOpen-LibrariesAndSDKs/AMF)
 * OpenCL で音声信号処理を行なう [TrueAudio Next (TAN)](https://github.com/GPUOpen-LibrariesAndSDKs/TAN)
 * Mesa3D とは別に AMD公式が提供するオープンソースな Vulkanドライバー、[AMDVLK](https://github.com/GPUOpen-Drivers/AMDVLK)
    * それを構成する [Vulkan® API Layer (XGL)](https://github.com/GPUOpen-Drivers/xgl)、[LLVM-Based Pipeline Compiler (LLPC)](https://github.com/GPUOpen-Drivers/llpc)、[Platform Abstraction Library (PAL)](https://github.com/GPUOpen-Drivers/pal)

等を提供してきた実績がある。  

### オープンソース……?
しかし GPUOpen と言っても、上記すべてがオープンソースという訳ではなく、[Radeon™ GPU Profiler](https://github.com/GPUOpen-Tools/radeon_gpu_profiler)、それとつい最近リリースされた[Radeon™ Memory Visualizer](https://github.com/GPUOpen-Tools/radeon_memory_visualizer)はバイナリのみのクローズドソースで提供されている。  
{{< link >}}[Plans to make source available ? · Issue #1 · GPUOpen-Tools/radeon_gpu_profiler](https://github.com/GPUOpen-Tools/radeon_gpu_profiler/issues/1){{< /link >}}

これは理由として、ライブラリやSDKよりもオープンソースで提供する意義が薄く、AMD も企業として活動する以上、隠しておきたい情報が含まれるといったことが考えられる。  
それでも GPUOpen は GPUOpen であり続けてきた。  

## 再度オープンソースに
RadeonRays 4.0 がクローズドソースになることがユーザー間で話題となり、"炎上"が巻き起こった後にAMD は動いた。  

RadeonRays 4.0 のリリースから2日後に、開発者の yozhijk(Dmitry Kozlov)氏は、社内で話し合った結果として、再度オープンソースにするとコメントした。  

 > Hi guys,
 > Thanks for still being with us and for the feedback you have provided so far. It has been decided yesterday to open-source most of RadeonRays4 and put AMD specific IP in closed-source modules (source will be available upon request under SLA).

 > -Dmitry

 > 引用元: <cite>[https://github.com/GPUOpen-LibrariesAndSDKs/RadeonRays_SDK/issues/206](https://github.com/GPUOpen-LibrariesAndSDKs/RadeonRays_SDK/issues/206)</cite>

これでめでたしめでたしとは思わない。  

AMD はユーザーからの反応に良くも悪くも敏感な所がある。  
2018年年初、最新ドライバーを導入すると DirectX 9 を用いるゲームが動作しなくなる問題がコミュニティで上げられ、話題となった。  
最初、AMD のエンジニアはこの問題に対し、「そのような古い(10年近く前)ゲームにエンジニアリングリソースを割くことはない」、と発言した。  
それが炎上を呼び、後にドライバー担当責任者はバグを修正することを明言し、実際に修正されたバージョンがリリースされた。  
{{< link >}}参考: <cite>[古いDX9タイトルが動作しない問題にひとまずの対処を行った「Radeon Software Adrenalin Edition 18.1.1 Alpha」が登場 - 4Gamer.net](https://www.4gamer.net/games/022/G002212/20180105001/)</cite>{{< /link >}}

この件は、ゲーマーという立場で直面した問題であり、DirectX 9 ゲームを AMD GPU環境でプレイしたいユーザーにとってはクリティカルな問題であった。  
問題の解決は歓迎すべきことだろう。  
それと同時に、古いゲームのサポートの代わりに新しいゲームへの最適化が進むとなれば、それもまたユーザーにとって悪い話ではないはずだ。  

ただ、ユーザーからの反応を見て、どうするのが得か AMDは考え、行動に移した。  
今回もそうだろうか？ いやユーザー、ゲーマーへの直接的な影響は限りなく小さかったはずだ。  

かつて、Linux Kernel の開発者であり優しい終身の独裁者を務める Linus Torvalds氏は NVIDIA に対し中指を立てた。  
これは Linuxユーザーからの「NVIDIA はオープンソースドライバー出してくれず、ドキュメントも整備されていないが、あなたはどう思う？」という質問に対し、開発者として回答したものだ。  
Linus Toravalds氏のパフォーマンスとも呼べる行動は話題となり、それから NVIDIA は態度を改め(?)、サポートを強化する旨の声明を出し[^1]、2019年にはドライバーの開発に必要な GPU の BIOS情報、電力管理機能やディスプレイ周りの情報を公開する、[NVIDIA open-gpu-doc repository](https://github.com/NVIDIA/open-gpu-doc)を立ち上げている。  

[^1]: [NVIDIAがLinux開発者リーナスのFワード発言を受けて今後の方針を発表 - GIGAZINE](https://gigazine.net/news/20120620-nvidia-respond-tovalds/)

これは Linuxユーザーの、そして開発者にとっての問題だったと判断できる。  

しかし、今回の件はどういった立場のユーザーにとっての問題だろう。そしてどういったユーザーに得となるのだろう。  
RadeonRays を利用したソフトウェアを開発している開発者だろうか？  
それに対しては、AMD のあるスタッフが関連した Reddit のスレッドにて、問題なく開発者は利用し続けられるとコメントしている。そして旧バージョンは引き続き MITライセンスの元、オープンソースで公開されるとも。  
{{< link >}}<https://www.reddit.com/r/Amd/comments/gkbsoy/it_appears_that_amd_is_going_to_open_source_most/fqqm78b?utm_source=share&utm_medium=web2x>{{< /link >}}
それに Githubレポジトリでリリースされるため、開発者への質問は直接、オープンで行なえる。  
変わらず、GPUOpen であると言えるはずだ。（これは冗談のつもりです）

自分なりの結論としては、オープンソースであることに対する AMD とユーザーの評価のギャップだったと考えている。  

[GPUOpen 内の Radeon Rays のページ](https://gpuopen.com/radeon-rays/)で、バージョン 2.0 と 4.0 の比較で、Open Souceの項目に 2.0 では良い印象を与える黄緑色のチェックマークを、4.0 には悪い印象を与える赤色のバツマークを付けている。  
このことから、AMD はオープンソースであることがアピール点になると考えていることがわかる。  
ただ、どれだけのアピールとなるか、なっていたかの評価が、ユーザーに比べると小さかったのだ。  
もしかしたらユーザーは具体的にどういった恩恵を享受できるかに関わらず、オープンソースであること自体を評価していたかもしれない。  
そして、ユーザーは評価していたオープンソース・ソフトウェアがクローズドソースになるという報を聞き、  
今後同様に他のソフトウェアがクローズドソースになることを危惧した。  
それが反感へと繋がり、"炎上"が起きることとなった。  

<br>
これからも GPUOpen は"変わらず"オープンであり続けることだろう。  
しかし、一度オープンソースにしたことで企業としての理屈に則る一貫した行動を取りにくくなることは、AMD にちょっとした後悔と方針の変化を与えることとなるかもしれない。  

{{< ref >}}

 * [Polaris アーキテクチャー | AMD](https://www.amd.com/ja/technologies/polaris)
 * [2012年6月18日　Linusが吠えた! ─中指立てて「NVIDIAは世界最悪の企業」：Linux Daily Topics｜gihyo.jp … 技術評論社](https://gihyo.jp/admin/clip/01/linux_dt/201206/18)
 * [AMD Rethinks Decision And Will Open-Source Most Of Radeon Rays 4.0 - Phoronix](https://www.phoronix.com/scan.php?page=news_item&px=Radeon-Rays-4.0-Going-Open)
 * [古いDX9タイトルが動作しない問題にひとまずの対処を行った「Radeon Software Adrenalin Edition 18.1.1 Alpha」が登場 - 4Gamer.net](https://www.4gamer.net/games/022/G002212/20180105001/)

{{< /ref >}}
