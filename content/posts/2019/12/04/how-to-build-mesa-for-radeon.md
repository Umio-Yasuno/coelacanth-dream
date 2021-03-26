---
title: "Radeon向けMesaビルド方法　―― RadeonSI, RADV"
date: 2019-12-04T21:00:00+09:00
draft: false
tags: [ "Radeon", "RadeonSI", "Mesa" ]
# keywords: [ "Mesa", "RadeonSI", "OpenGL" ]
categories: [ "Software", "AMD", "GPU" ]
---

<https://dri.freedesktop.org/wiki/Building/> を元に、なんやかんやである程度確立させたMesaのビルド方法をまとめてみた。  
自環境がDebianなため、基本Debian/Ubuntu向けとなる。  

## インデックス

 * [libdrm](#libdrm)
 * [Mesa](#mesa)

## libdrm

[RadeonSIでOpenGL 4.6がサポート](/posts/2019/11/29/radeonsi-enable-gl4_6)でも書いたが、最新版のMesaをビルドするにはバージョン2.4.99以上のlibdrmが必要となる。  
厳密には、mesa-19.2.6では**2.4.99**、git、mesa-19.3.0-rc5では**2.4.100**だが、どうせなら最新版の**2.4.100**をビルド、インストールしてしまった方がいい。  
リリースページからダウンロードしてもいいが、アップデートに追従することを考えると gitレポジトリをクローンした方がいい。  

 * [Mesa / drm · GitLab](https://gitlab.freedesktop.org/mesa/drm)

Debian/Ubuntu では以下のコマンドで libdrm のビルドに必要なパッケージをインストールできる。  

	# apt build-dep libdrm

そしてビルドツールだが、Meson、Ninja が推奨となっている。これはMesaも同じであるため、どちらも同じビルドツールを使った方が楽。  
Mesonの使い方は簡単で、

	$ meson [適当なディレクトリ名]

とやるだけで準備は済む。  
ビルド時間短縮やインストールサイズの削減のため一部オプションを変更する、[適当なディレクトリ名]の後に -D を付けてからオプション名、= をやりオプションに応じた文字列/真偽値を入力する。  
今後、ここで生成したディレクトリを[ディレクトリ名]と記述する。  

オプションの一覧は、一度準備を済ませた後、

    $ meson configure [ディレクトリ名]

とすれば、オプションの状態と共に出力される。  
そのままでは見にくいため、

	$ meson configure [ディレクトリ名] > meson-configure.txt

のようにして一度テキストファイルに出力し、エディタ等で開くと見やすくなる。  

	オプションの変更例:
	$ meson reconfigure [ディレクトリ名] -Damdgpu=true -Dnouveau=false

libdrm はそこまで規模は大きくないため、わざわざ削る必要はあまりない。  
コンパイラの最適化設定は、 -Dc_args="[最適化フラグ]" 。  
ビルドの実行は以下のコマンドで。  

	$ ninja -C [ディレクトリ名] -j [並列数]

ビルドが完了したら、

	# ninja -C [ディレクトリ名] install

でインストールされる。  

## Mesa

Mesa も libdrm 同様ビルドに必要なパッケージを事前にインストールする。  

	# apt build-dep mesa

ただ、これでインストールされるのは、Debian/Ubuntuの公式レポジトリにて配布しているバージョンのMesaをビルドするのに必要なパッケージなため、最新版をビルドするには追加で必要なものもある。  

	$ meson [適当なディレクトリ名]

で準備はされるが、そのままだといろいろと問題が出てくる。  
コンパイラに gcc が使われたり、llvm-config が /usr/bin下のものになってたり、デバッグ機能が有効で性能が落ちたり、ビルドに必要以上に時間が掛かる。  

コンパイラにclang-Xを使うには（Xにはclangのバージョンを表す数字が入る）、`meson` コマンドの前に以下を入力することで環境変数に設定する。  

	$ CC="clang-X" CXX="clang++-X"

llvm-configをclangのバージョンと合わせるには、custom-llvm.ini といった適当ファイルを作成し、そこに以下のようにして使いたい `llvm-config` のパスを記述する。  

	[binaries]
	llvm-config = '/usr/local/bin/llvm/llvm-config'

そして `meson` 実行時に以下のオプションを加える。  

	--native-file custom-llvm.ini

 * [Compilation and Installation Using Meson - Mesa3D](https://www.mesa3d.org/meson.html#advanced)  

デバッグ機能は、オプションで buildtype=plain、b_ndebug=trueを追加することで無効化できる。  

ビルドされるものを減らすには、

	$ meson configure [ディレクトリ名] > meson-configure.txt

を見ながら、取捨選択することになる。Radeon向けの設定を最低限書くと以下のようになる。  

	-Ddri-drivers="r100,r200" -Dgallium-drivers="radeonsi,r300,r600" -Dvulkan-drivers="amd"

GCN以上の世代のRadeonを使っているのならradeonsiだけで良いかもしれない。  
あとはgallium-openclなんかも基本削る。  
現在は OpenCLを使う場合、Mesa とは 別に ROCm か AMDGPU-PROドライバーをインストールして方がいい。  

以上を踏まえると、`meson` の実行時オプションは以下のようになる。  

	CC="clang-X" CXX="clang++-X" meson [ディレクトリ名] -Dbackend=ninja -Dbuildtype=plain -Ddri-drivers="r100,r200" -Dgallium-drivers="radeonsi,r300,r600" -Dvulkan-drivers="amd" -Dgallium-opencl=disabled --native-file custom-llvm.ini

インストール場所（prefix）だが、<https://dri.freedesktop.org/wiki/Building/> にあるように、ホームディレクトリ下に適当なディレクトリを作成、 そしてそこにインストールし、各種環境変数を設定する。  
この方がトラブルを回避しやすく、気軽に最新バージョンの Mesaドライバー試せる。  
ただ、glxinfo や vainfo の結果を見るだけならスクリプトを書いたり、.bashrc の編集でいいのだが、それではデスクトップ上のメニューからアプリケーションを起動した際に反映されない。  
その場合 .xsessionrc に記述することで環境変数をデスクトップアプリケーションにも反映させることができる。  

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

記述するだけでは不十分であり、すぐに反映させたい場合は Xorg を再起動する必要がある。  
システムごと再起動させてもいいが、以下のようにディスプレイマネージャー (e.g. lightdm) を再起動させても、再ログインの必要が出るが反映させることはできる。  

	# systemctl restart [ディスプレイマネージャー]

