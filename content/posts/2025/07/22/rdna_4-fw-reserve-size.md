---
title: "Linux + RDNA 4 GPU で起動直後から VRAM 516MiB が使用済みになっている理由"
date: 2025-07-22T23:06:07+09:00
draft: false
categories: [ "AMD", "GPU" ]
tags: [ "RDNA_4", "GFX12", "Linux_Kernel" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

## TL;DR {#tldr}
Linux 環境における AMDGPU ドライバーは、VBIOS (ファームウェア) に記述された情報に応じて VRAM の一部を予約領域として割り当てるが、RDNA 4 GPU では何故かこれが 516MiB とかなり巨大なサイズの設定になっているため。  

## 経緯
タイトルの通り、Linux 環境で RDNA 4 GPU を使用すると起動直後から VRAM 520MiB 近くがすでに使用済みとなっている理由を調査したため、その記録。  
筆者は現在 RX 9060 XT 16GB を使用しており、起動直後かつ映像出力やコンポジタ等は iGPU が処理しているのにも拘らず VRAM 使用量が 520MiB 近いことに気付いてはいた。  
だが他に報告している人がいなかったこと、VRAM 16GB の GPU を使うのが初めてであったこと、そして最新世代の GPU なのだから不安定な部分はあるだろうとスルーしていた。  
だが drm/amd にこの問題に関する issue が投稿されたため、一度しっかりと調べることにした。  

 * [High vram allocation on gfx1201 (#4437) · Issue · drm/amd](https://gitlab.freedesktop.org/drm/amd/-/issues/4437)

## 調査
まず AMDGPU ドライバー側でいくつかの用途のために VRAM を割り当てることは知っていたため、それらしい部分のソースコードを読んだり、ログへの出力を追加してみたりした。  
結果的に空振りだったため、ここの部分の詳細は省く。  

次に VRAM のサイズを変えることで、この VRAM 使用量が変わるか、あるいはどのようなエラーが出るかを調べることにした。  
AMDGPU ドライバーのモジュールパラメータには `vramlimit` というものがあり、ここにサイズ (MiB) を設定することで AMDGPU ドライバーが認識する VRAM サイズを制限することができる。  

 * [Module Parameters — The Linux Kernel documentation](https://www.kernel.org/doc/html/latest/gpu/amdgpu/module-parameters.html)

最初に 1024 (MiB) を設定してみたが、特に変化はなかった。  
次に 512 (MiB)、AMDGPU ドライバー側の割り当てだけで制限を超えるサイズに設定してみたところ、`amdgpu_ttm_reserve_tmr` 関数でエラーが発生し、起動に失敗するようになった。  

`amdgpu_ttm_reserve_tmr` 関数の中身を読むと、ファームウェアから得られる情報に応じて、メモリトレーニングや IP 検出 バイナリ (IP discover binary)、その他様々な機能のために VRAM の一部を割り当てているようだった。  
そこで以下のようなパッチを適用し、割り当てるサイズをログに出力するように変更したところ、516MiB というサイズを割り当てていることがわかった。  

 >         diff --git a/drivers/gpu/drm/amd/amdgpu/amdgpu_ttm.c b/drivers/gpu/drm/amd/amdgpu/amdgpu_ttm.c
 >         index 9c5df35f0..b03fa58ff 100644
 >         --- a/drivers/gpu/drm/amd/amdgpu/amdgpu_ttm.c
 >         +++ b/drivers/gpu/drm/amd/amdgpu/amdgpu_ttm.c
 >         @@ -1772,6 +1772,8 @@ static int amdgpu_ttm_reserve_tmr(struct amdgpu_device *adev)
 >          	else if (!reserve_size)
 >          		reserve_size = DISCOVERY_TMR_OFFSET;
 >          
 >         +	dev_info(adev->dev, "reserve size %uM\n", reserve_size >> 20);
 >         +
 >          	if (mem_train_support) {
 >          		/* reserve vram for mem train according to TMR location */
 >          		amdgpu_ttm_training_data_block_init(adev, reserve_size);

この 516MiB というサイズは予約済み領域、言い換えればユーザーが使用できない領域としては大きく、現状 RDNA 4 GPU で最も廉価である RX 9060 XT 8GB では他モデル (RX 9060 XT 16GB, RX 9070 /XT 16GB) よりも性能面で影響を強く受けるだろう。  
またこれは従来の世代よりもずっと大きく、RX 6600 8GB の場合に割り当てられるサイズはたったの 3MiB である。  
drm/amd に issue を投稿した Adriano Martins 氏によれば、Windows 環境では起動直後で 80MiB のみが使用済みとされるらしいため、今後修正されることを願っている。  

## 他の確認方法
もし Linux Kernel への変更なしで予約されるサイズを調べたい場合は、以下のリポジトリをクローンし、`cargo run --example vbios_parser | grep fw_reserved_size_in_kb` を実行することで確かめられる。  
以下は自作のリポジトリとツールであり、RX 6600 8GB, Ryzen 5 9600X, RX 9060 XT 16GB で動作を確かめてはいるが、正常な結果を返せない場合もあるかもしれない点は留意していただきたい。  

 * <https://github.com/Umio-Yasuno/libdrm-amdgpu-sys-rs>
