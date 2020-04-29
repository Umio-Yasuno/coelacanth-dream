---
title: "waifu2x-ncnn-vulkan ビルド方法(Linux向け) 【2020/04/29 追記修正】"
date: 2020-04-26T07:53:15+09:00
draft: false
# tags: [ "", ]
keywords: [ "waifu2x-ncnn-vulkan", ]
categories: [ "Software", "GPU" ]
noindex: false
---

Vulkan Computeで演算するため、Linux + Radeon 環境でもローカルで実行可能な画像高解像度ソフトウェア、waifu2x-ncnn-vulkan をビルドする方法のメモ。

 * [ncnn のビルド方法](#ncnn)
 * [waifu2x-ncnn-vulkan のビルド方法](#waifu2x-ncnn-vulkan)

## ncnn のビルド方法 {#ncnn}
waifu2x-ncnn-vulkan をビルドするには、まず ncnn をビルドする必要があるのだが、  
<del>最新版ではビルド時にエラーが出るため、ncnn を `git clone` した後、waifu2x-ncnn-vulkan のページにあるリンクのコミット時に切り替えるのがいい。  
{{< link >}}<https://github.com/Tencent/ncnn/wiki/how-to-build#build-for-linux-x86>{{< /link >}}

	$ git clone https://github.com/Tencent/ncnn
	$ cd ncnn
	$ git checkout 8c537069875a28d5380c6bdcbf7964d73803b7b3
</del>

{{< ins datetime="2020-04-29" >}}
最新版ではエラーが出ると書いたが、原因が発覚した。  
端的に言えば `glslangValidator` が最新版でなかった。誰かのせいにするなら cmake が `VULKAN_SDK` を認識してくれないのが悪い。  
そういうことで以下修正版。

	$ cmake -DVulkan_LIBRARY="${VULKAN_SDK}/lib/libvulkan.so" -DVulkan_INCLUDE_DIR="${VULKAN_SDK}/include" -DNCNN_VULKAN=ON -DGLSLANGVALIDATOR_EXECUTABLE=${VULKAN_SDK}/bin/glslangValidator ../
	$ make -j$(grep -c processor /proc/cpuinfo)
	$ make install

インストール先はデフォルトで `<build dir>/install`

{{< /ins >}}

<del>ncnn の wiki には VulkanSDK をダウンロード、展開し、環境変数 `VULKAN_SDK` をそこにセットするようにとあるが、CMake がそれを見つけてくれないため、CMake 実行時、別に指定する必要がある。</del>  

<del>まずビルド用のディレクトリを作成。下記のコマンドはディレクトリ名に日付を入れるようにしているが、名前であるから何でもいい。  
そして作成したディレクトリに移動。</del>

	$ mkdir build_$(date "+%F")
	$ cd <build dir>

<del>
cmake、ビルド、そしてインストール。インストール先はデフォルトで `<build dir>/install` になっている。</del>  

	$ cmake -DVulkan_LIBRARY="${VULKAN_SDK}/lib/libvulkan.so" -DVulkan_INCLUDE_DIR="${VULKAN_SDK}/include" -DNCNN_VULKAN=ON ../
	$ make -j$(grep -c processor /proc/cpuinfo)
	$ make install

## waifu2x-ncnn-vulkan のビルド方法 {#waifu2x-ncnn-vulkan}
waifu2x-ncnn-vulkan のバージョンは問わず、最新版でも問題ないが、ここでは version 20200414 を前提とする。  
{{< link >}}[Release version 20200414 · nihui/waifu2x-ncnn-vulkan](https://github.com/nihui/waifu2x-ncnn-vulkan/releases/tag/20200414){{< /link >}}

まずは waifu2x-ncnn-vulkan-20200414 の src ディレクトリへ。  
そして先程同様にビルド用のディレクトリを作成、移動。

	$ cd waifu2x-ncnn-vulkan-20200414/src
	$ mkdir build_$(date "+%F")
	$ cd <build dir>

cmake を実行前に、CMakeList.txt を編集する必要があり、67行目の一部を先程ビルドした ncnn に合わせて変更する。  
パスはその人の環境で変わる。下は、/home/${USER}/src 下に ncnn をクローン、ビルドした場合のパス。  

	/home/${USER}/src/<build dir>/install/lib/cmake/ncnn

<del>そしてビルド。ここでは特にオプションは必要としない。</del>

{{< ins datetime="2020-04-29" >}}

ここでも cmake で Vulkan系を指定する必要がある。  

	$ cmake -DVulkan_LIBRARY="${VULKAN_SDK}/lib/libvulkan.so" -DVulkan_INCLUDE_DIR="${VULKAN_SDK}/include" -DGLSLANGVALIDATOR_EXECUTABLE=${VULKAN_SDK}/bin/glslangValidator ../src
	$ make -j$(grep -c processor /proc/cpuinfo)

{{< /ins >}}

	$ cmake ../
	$ make -j$(grep -c processor /proc/cpuinfo)

`<build dir>` に waifu2x-ncnn-vulkan のバイナリが生成されるため、それを実行すればいいが、推論に用いるモデルは `<build dir>` 下ではなく、waifu2x-ncnn-vulkan 下にあるため注意。  

ビルドに失敗する時は、以下で質問して頂ければ助けになれるかもしれません。  
{{< link >}}[Issues · Umio-Yasuno/umio-yasuno.github.io](https://github.com/Umio-Yasuno/umio-yasuno.github.io/issues){{< /link >}}
