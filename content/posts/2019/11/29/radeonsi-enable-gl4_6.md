---
title: "RadeonSIでOpenGL 4.6がサポート"
date: 2019-11-29T06:21:42+09:00
draft: false
tags: [ "Radeon", "RadeonSI", "Mesa" ]
keywords: [ "Mesa", "RadeonSI", "OpenGL" ]
categories: [ "Software", "AMD", "GPU" ]
---

オープンソースなRadeon OpenGLドライバ、RadeonSIにてOpenGL 4.6が有効「可能」となった。  
[radeonsi: enable SPIR-V and GL 4.6 for NIR](https://gitlab.freedesktop.org/mesa/mesa/commit/754c7b893959d97483e6b5fccefbdbaa641c70ca)  
しかしデフォルトでは有効にならず、ひと手間設定を変える必要がある。  

### 有効方法
まだ正式にはリリースされていないため、Debian/Ubuntuユーザーは以下のようなPPA（Personal Package Archives）を追加、そして最新版をインストール、  
<https://launchpad.net/~oibaf/+archive/ubuntu/graphics-drivers>  
もしくは[Mesaのmasterブランチ](https://gitlab.freedesktop.org/mesa/mesa)からクローン、そして自前でビルドをする必要がある。  

上記PPAはほんと最新版を追い続けるため、アップデートがしつこい。使用していた頃、ほぼ毎日来ていたと記憶している。  
リリース候補版でもないため、不具合が起きやすい。それなのに、Debian/Ubuntuの正式パッケージを（たぶん）上書きするため、  
致命的な不具合があった場合、復旧がちと面倒だ。  
そのため、自前でビルドすることをオススメしたい。  
ただ、ビルドの準備をするのも結構な手間がかかる。  
libdrm_amdgpuのバージョンが2.4.99以上を要求されるため、ディストリビューションによってはまずそちらの最新版をビルドする必要があったり、  
[meson: require libdrm_amdgpu 2.4.99 for Navi](https://gitlab.freedesktop.org/mesa/mesa/commit/677bb80c98276514d3df497d3c88908158794637)  
[[ANNOUNCE] libdrm 2.4.100](https://lists.x.org/archives/xorg-announce/2019-October/003028.html)  
RadeonSI、RADVにてLLVM 7のサポートがなくなったため、これもディストリによってはLLVM 8/9をインストールする必要がある。
[ac,radv,radeonsi: remove LLVM 7 support](https://gitlab.freedesktop.org/mesa/mesa/commit/1fd60db4a1fca96ccf9293d0c03158baf7d215a5)  

軽く調べたところ、

| Distribution | LLVM | libdrm_amdgpu |
| :--- | :---: | :---: |
| Debian buster (stable) | 7 | 2.497 |
| Debian bullseye (testing) | 8 | 2.4.100 |
| Ubuntu bionic (18.04LTS) | 6 | 2.492 |
| Ubuntu disco (19.04) | 8 | 2.497 |

となっていた。  
Debian bullseye (testing)以外では別途インストールする必要がある。  
Debian/UbuntuでのLLVM 8/9のインストールは下記URLに従えばいいだけなため、特別難しくはない。  
[LLVM Debian/Ubuntu nightly packages](https://apt.llvm.org/)  

Mesaの詳細なビルド方法は長くなるだろうから、私が参考にしたURLを載せておく。  
<https://dri.freedesktop.org/wiki/Building/>  

前置きが少し長くなってしまった。  
そしてNIRの有効方法だが、一番楽なのは .drirc を編集することだ。  
driconf、 [adriconf](https://github.com/jlHertel/adriconf) を使うか、  
直接 ~/.drirc を編集して、 Default の項目内に

	<option name="radeonsi_enable_nir" value="true" />

を追加すればいい。  
試したいだけなら、OpenGLを呼び出すコマンド（glxinfo -B等）の前に  

	radeonsi_enable_nir=true

を入力してもいい。  

### OpenGL 4.6
OpenGL 4.6がリリースされたのは2017年のSIGGRAPH会期中であり、だいぶ前である。  
[OpenGL 4.6の進化点やOpenCLの将来について，Khronos Group代表のNeil Trevett氏に聞いてみた - 4gamer.net](https://www.4gamer.net/games/107/G010729/20170907023/)  
だが、Windows環境やプロプライエタリなドライバでは検証、登録される中、Linux&オープンソースドライバでは来るか来るかと言われながら、  
つい最近になってようやくIntel、AMD共に公式にサポートされた。  
<https://www.khronos.org/conformance/adopters/conformant-products/opengl>  

[[Mesa-dev] iris now officially OpenGL 4.6 conformant](https://lists.freedesktop.org/archives/mesa-dev/2019-November/223795.html)  
[[Mesa-dev] [radeonsi] enable OpenGL 4.6](https://lists.freedesktop.org/archives/mesa-dev/2019-November/223821.html)  

OpenGL 4.6になってユーザーが何か得するのかと言うと、一応はすると思う。  
[クロノス・グループ、SPIR-V機能を搭載した「OpenGL® 4.6」を発表](https://jp.khronos.org/news/press/spir-v-opengl-4.6-siggraph-2017)  
RadeonSIではNIRを有効にする必要があり、そしてNIRでは少し前の結果ではあるものの、若干の性能向上とCPU使用率の減少が確認されている。  
[https://www.phoronix.com/scan.php?page=article&item=radeonsi-nir-2019](https://www.phoronix.com/scan.php?page=article&item=radeonsi-nir-2019&num=1)  
今回のOpenGL 4.6サポートはこのNIRが安定したということでもあるため、このことによる得も享受できる。  
ディスクキャッシュ周りも改良が続けられており、それがおかげが「体感で」快適になったとも感じる。  
あくまで体感だから信用はならないが。  

あ、あと libzstd-dev のインストールを忘れずに。  
