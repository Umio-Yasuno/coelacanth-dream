---
title: "Radeon向けMesaビルド方法　―― RadeonSI, RADV"
date: 2019-12-04T21:00:00+09:00
draft: false
tags: [ "Radeon", "RadeonSI", "Mesa" ]
# keywords: [ "Mesa", "RadeonSI", "OpenGL" ]
categories: [ "Software", "AMD", "GPU" ]
---

<https://dri.freedesktop.org/wiki/Building/>
を元に、なんやかんやである程度確立させたMesaのビルド方法をまとめてみた。  
私の環境がDebianなため、基本Debian/Ubuntu向けとなる。  

**このサイトを参考にコマンドを実行し、何かしらの損害が発生したとしても筆者は責任は持たない**ことを留意していただきたい。  

## インデックス

 * [libdrm](#libdrm)
 * [Mesa](#mesa)

## libdrm
[RadeonSIでOpenGL 4.6がサポート](/posts/2019/11/29/radeonsi-enable-gl4_6)でも書いたが、最新版のMesaをビルドするにはバージョン2.4.99以上のlibdrmが必要となる。  
厳密には、mesa-19.2.6では**2.4.99**、git、mesa-19.3.0-rc5では**2.4.100**だが、  
どうせなら最新版の**2.4.100**をビルド、インストールしてしまった方がいい。  

まず、下記リンク内からダウンロード、展開する。  
[[ANNOUNCE] libdrm 2.4.100](https://lists.x.org/archives/xorg-announce/2019-October/003028.html)  

次に以下のコマンドでlibdrmのビルドに必要なパッケージをインストールする。  

	# apt build-dep libdrm

そしてビルドツールだが、Meson、Ninjaが推奨となっている。  
これはMesaも同じだ。  

Mesonの使い方は簡単で、

	$ meson [適当なディレクトリ名]

とやるだけで準備は済む。  
オプションを変えるには、[適当なディレクトリ名]の後に -D を付けてからオプション名、= をやりオプションに応じた文字列/真偽値を入力する。  
今後、ここで生成したディレクトリを[ディレクトリ名]と記述する。  
オプションの一覧は、一度準備を済ませた後、

	$ meson configure [ディレクトリ名]

とすれば、オプションの状態と共に出力される。  
そのままでは見にくいため、

	$ meson configure [ディレクトリ名] > meson-configure.txt

のようにして一度テキストファイルに出力し、エディタ等で開くと見やすくなる。  

	オプションの変更例:
	$ meson reconfigure [ディレクトリ名] -Damdgpu=true -Dnouveau=false

そこまでビルドの規模は大きくないため、わざわざ削る必要はあまりなかったりする。  
コンパイラの最適化設定は、 -Dc_args="[最適化フラグ]" 。  
Debug周りがそのままだと有効にされ、性能が落ちると言われるが、libdrmでは気にする必要はない、と思う。  
肝心のビルドをするには、

	$ ninja -C [ディレクトリ名] -j [並列数]

そしてビルドが完了したら、

	# ninja -C [ディレクトリ名] install

でインストールされる。  

## Mesa
これも事前に

	# apt build-dep mesa

ただ、これでインストールされるのは、Debian/Ubuntuの公式レポジトリにて配布しているバージョンのMesaをビルドするのに必要なパッケージなため、  
最新版をビルドするには追加で必要なものもある。  
必須という訳ではないが、 libzstd-dev がシェーダーキャッシュに使われるためインストールするのをオススメする。  
libdrm同様、

	$ meson [適当なディレクトリ名]

で準備はされるが、そのままだといろいろと問題が出てくる。  
コンパイラにgccが使われたり、llvm-configが/usr/bin下のものになってたり、デバッグ機能が有効で性能が落ちたり、ビルドされるものが多く、時間がかかったり。  
これらの解決方法は簡潔に書く。  

コンパイラにclang-Xを使うには、（Xにはclangのバージョンを表す数字が入る）

	$ CC="clang-X" CXX="clang++-X"

llvm-configをclangのバージョンと合わせるには、custom-llvm.iniといったファイルを作成し、そこへ

	[binaries]
	llvm-config = '/usr/local/bin/llvm/llvm-config'

のようにしてllvm-configのパスを記述、mesonの実行時に、

	--native-file custom-llvm.ini

のようにする。  
[Compilation and Installation Using Meson - Mesa3D](https://www.mesa3d.org/meson.html#advanced)  

デバッグ機能を無効にするには、オプションで buildtype=plain、b_ndebug=trueを追加する。-Dを忘れずに。  

ビルドされるものを減らすには、

	$ meson configure [ディレクトリ名] > meson-configure.txt

を見ながら、取捨選択する……というのは投げやり過ぎるのでRadeon向けの設定を最低限書くと、

	-Ddri-drivers="r100,r200" -Dgallium-drivers="radeonsi,r300,r600" -Dvulkan-drivers="amd"

となる。GCN以上の世代のRadeonを使っているのならradeonsiだけで良いかもしれない。  
あとはgallium-openclなんかも基本削る。  
OpenCLを使いたいなら、Mesaとは別にROCmかAMDGPU-PROをインストールしてRadeonを活かした方がいい。  

ここまでを踏まえると、このようになる。  

	CC="clang-X" CXX="clang++-X" meson  [ディレクトリ名] -Dbackend=ninja -Dbuildtype=plain -Ddri-drivers="r100,r200" -Dgallium-drivers="radeonsi,r300,r600" -Dvulkan-drivers="amd" -Dgallium-opencl=disabled --native-file custom-llvm.ini

それでもそれなりに時間はかかるため、気長に待とう。  
コンパイラへの最適化フラグ等には特に触れない。  

インストール場所（prefix）だが、<https://dri.freedesktop.org/wiki/Building/>にあるように、ホームディレクトリ下に適当なディレクトリを作成、  
そしてそこにインストール、環境変数設定の方がトラブルを回避しやすく、気軽に試せる。  
ただ、glxinfo や vainfo の結果を見るだけならスクリプトを書いたり、.bashrcの編集でいいのだが、それではデスクトップ上のメニューからアプリケーションを起動した際に反映されない。  
すべてのアプリをターミナルから起動する訳にもいかないので、私は .xsessionrc をに記述するようにしている。  
実際には、

	PROJECT="[ホーム]/gfx/11-29-0351_20.0.0-devel"
	PROJECT_LIB=$PROJECT/lib/x86_64-linux-gnu
	export LD_LIBRARY_PATH=$PROJECT_LIB:$LD_LIBRARY_PATH
	export PKG_CONFIG_PATH=$PROJECT_LIB/pkgconfig:$PKG_CONFIG_PATH
	export LIBGL_DRIVERS_PATH=$PROJECT_LIB/dri:$LIBGL_DRIVERS_PATH
	export EGL_DRIVERS_PATH=$PROJECT_LIB:$EGL_DRIVERS_PATH
	export LIBVA_DRIVERS_PATH=$PROJECT_LIB/dri:$LIBVA_DRIVERS_PATH
	export VK_ICD_FILENAMES=$PROJECT/share/vulkan/icd.d/radeon_icd.x86_64.json:$VK_ICD_FILENAMES
	export PATH=$PROJECT/bin:$PATH
	LIBVDPAU_DRIVERS_PATH=$PROJECT_LIB/vdpau:$LIBVDPAU_DRIVERS_PATH

となる。こうすればFirefoxといったアプリにもちゃんと反映される。  
Mesaビルド一連の流れはbash シェルスクリプトで実行してるが、人様に見せられるようなクオリティではないので公開は控える。  

とはいえこれでもまだ不十分であり、Xorgを再起動させる必要がある。  
システムごと再起動させるのはだるいので、私は

	# systemctl restart [ディスプレイマネージャー]

としている。  

最後に、最新版のMesaを自力でビルドすることの意義だが、最新の機能、ハードウェアを試せるというロマンに尽きると思う。  
