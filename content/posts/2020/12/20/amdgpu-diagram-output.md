---
title: "AMD GPU のダイアグラムを出力するスクリプトを作った話"
date: 2020-12-20T16:50:16+09:00
draft: false
# tags: [ "", ]
# keywords: [ "", ]
categories: [ "Diary", ]
noindex: false
# summary: ""
toc: false
---

気が付けばもう 2020年がほとんど残っていなかったので雑記、くだらないことを書きたくなった。  

タイトルの通り、AMD GPU のダイアグラムを出力するシェルスクリプトを作った。  

 * [Umio-Yasuno/amdgpu-diagram-output](https://github.com/Umio-Yasuno/amdgpu-diagram-output)

実行例は以下みたいな感じ。  

{{< dia >}}

```
## RX 560

 +- ShaderEngine(00) -----------------+  +- ShaderEngine(01) -----------------+ 
 | +- ShaderArray(00) --------------+ |  | +- ShaderArray(00) --------------+ | 
 | | ====  ====  CU (00) ====  ==== | |  | | ====  ====  CU (00) ====  ==== | | 
 | | ====  ====  CU (01) ====  ==== | |  | | ====  ====  CU (01) ====  ==== | | 
 | | ====  ====  CU (02) ====  ==== | |  | | ====  ====  CU (02) ====  ==== | | 
 | | ====  ====  CU (03) ====  ==== | |  | | ====  ====  CU (03) ====  ==== | | 
 | | ====  ====  CU (04) ====  ==== | |  | | ====  ====  CU (04) ====  ==== | | 
 | | ====  ====  CU (05) ====  ==== | |  | | ====  ====  CU (05) ====  ==== | | 
 | | ====  ====  CU (06) ====  ==== | |  | | ====  ====  CU (06) ====  ==== | | 
 | | ====  ====  CU (07) ====  ==== | |  | | ====  ====  CU (07) ====  ==== | | 
 | |  [ RB ] [ RB ]                 | |  | |  [ RB ] [ RB ]                 | | 
 | |  [ Rasterizer/Primitive Unit ] | |  | |  [ Rasterizer/Primitive Unit ] | | 
 | +--------------------------------+ |  | +--------------------------------+ | 
 |      [- Geometry Processor -]      |  |      [- Geometry Processor -]      | 
 +------------------------------------+  +------------------------------------+ 
    [L2$ 256K]      [L2$ 256K]      [L2$ 256K]      [L2$ 256K]      
```

{{< /dia >}}

機能はシンプルで、**RadeonSI (OpenGL)** ドライバーのデバッグ機能から得られる情報を元にそれっぽいダイアグラムを出力する。  

作った動機は、**RX 560** で CU数を変えて性能への影響を確かめる検証を行なった時、ShaderEngine、ShaderArray、CU がそれぞれどのような位置にあるかを画像で示したが、これが単純に面倒くさかった。ささっと軽く作ったが、センスの無さもあってチープなのも気に入らない。  
{{< link >}} [RX 560 で CU数が GPU性能にどれだけ影響を与えるかを調査 【性能編】 | Coelacanth's Dream](/posts/2020/08/06/polaris11-cu-scaling-test/#disable_cu) {{< /link >}}
そうしたことがあって、**RX 6800** について書いた時は画像を作らなかったが、やはり図があった方が説明も理解もしやすい。  
{{< link >}} [AMD GPU としては珍しい構成の Radeon RX 6800 | Coelacanth's Dream](/posts/2020/11/13/amd-rx-6800/) {{< /link >}}

実際には持ってない AMD GPU の解説にも使えるよう、一部情報を上書きするオプションも付けてある。  
テキストダイアグラムを使うことで、作成に掛ける時間を大幅に短縮しつつデータの転送量を抑えることができる。  
問題点はフォントに左右されることで、ダイアグラムの描画には基本的な記号を用いているから一般的な等幅フォントであれば問題ないはずだが、ブラウザ環境は種々雑多であるからそう言い切れない。  
それを回避しようと Imagemagick で画像を出力する機能も持たせようとしたが、上の実行例ぐらい小さいダイアグラムでは問題なく綺麗に出力されるが、大きくなると失敗する不具合があったため、今の所はコメントアウトしてある。成功した時の画像サイズから、設定されている最大サイズは超えないはずなのだが、エラーメッセージは超えていると言ってくる。  


自分以外に需要があるかは怪しいけど、一応下記のように各情報とピーク性能を出力する機能も持たせてる。  


 >        Driver Version:		Mesa 21.0.0-devel (git-b9fccafed6)
 >        
 >        GPU ASIC:		POLARIS11
 >        Chip class:		GFX8
 >        Marketing Name:		Radeon RX 560 Series
 >        GPU Type:		Discrete GPU
 >        
 >        Compute Units:		  16 CU
 >        GFX Clock Range:	 214 MHz - 1080 MHz
 >        Peak GFX Clock:		1196 MHz
 >        
 >        Peak FP16:		 2.44 TFlops
 >        Peak FP32:		 2.44 TFlops
 >        
 >        RBs (Render Backends):		  4 RB (16 ROP)
 >        Peak Pixel Fill-Rate:		 19.13 GP/s
 >        TMUs (Texture Mapping Units):	 64 TMU
 >        Peak Texture Fill-Rate:		 76.54 GT/s
 >        
 >        VRAM Type:		    GDDR5
 >        VRAM Size:		  4096 MB
 >        VRAM Bit Width:		   128-bit
 >        Memory Clock Range:	   300 MHz - 1750 MHz
 >        Peak Memory Clock:	  1750 MHz
 >        Peak VRAM Bandwidth:	   112.00 GB/s
 >        
 >        L2 Cache Blocks:	  4 Block
 >        L2 Cache Size:		  1 MB (1024 KB)
 >        
 >        Power cap:		 48 W
 >        
 >        Card Interface:		PCIe Gen3 x8 
 >        
 >        AMD Smart Access Memory

今まさに画面を描画している AMD GPU のスペックと、ダイアグラムによって大体の構造を知り、興味を持つきっかけとなってくれたら幸いだ。  
