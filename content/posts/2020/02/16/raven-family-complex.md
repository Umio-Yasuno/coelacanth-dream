---
title: "いかに Zen APU は複雑か"
date: 2020-02-16T02:32:47+09:00
draft: false
tags: [ "Raven", "Renoir", "Picasso", "Raven2", "Dali", "Pollock" ]
keywords: [ "", ]
categories: [ "Hardware", "APU" ]
noindex: false
---

先日、Chromium OSへのパッチを眺めていたら、Raven2 (Dali, Pollock)を判定するコードの関数名の一部で *zen2* が使われていた。  
[soc/amd/picasso: Add helper functions for finding SOC type (I24b73145) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2051514)  

当然これは間違いであり、Raven2 (Dali, Pollock)のCPUコアは、  
EPYC 7002シリーズ (Rome)、Ryzen 3000シリーズ (Matisse)、Ryzen Threadripper 3000シリーズ (Castle Peak)、Ryzen Mobile 4000シリーズ (Renoir)に採用されている、 *あの* Zen2ではない。  

AMDもはっきりと、Raven2ベースの製品、Athlon 3000Gを発表の際に "Zen" ベースであると言っているし、  

 >  The Athlon 3000G is the first “Zen”-based Athlon processor that is unlocked for overclocking potential, delivering the only unlocked processor in its segment10.

 > 引用元: [AMD Introduces World’s Most Powerful 16-core Consumer Desktop Processor, the AMD Ryzen™ 9 3950X | Advanced Micro Devices](https://ir.amd.com/news-releases/news-release-details/amd-introduces-worlds-most-powerful-16-core-consumer-desktop)  

Raven2 (Dali)ベースのAthlon Gold 3150U、Athlon Silver 3050Uでも "Zen" アーキテクチャだとしている。  

 > Bringing consumers more choice, the new AMD Athlon 3000 Series Mobile Processor family expands the reach of the powerful “Zen” architecture to mainstream notebooks.

 > 引用元: [AMD Announces World’s Highest Performance Desktop and Ultrathin Laptop Processors at CES 2020 | Advanced Micro Devices](https://ir.amd.com/news-releases/news-release-details/amd-announces-worlds-highest-performance-desktop-and-ultrathin)

<br>
パッチ内のメッセージでもその点が突っ込まれまくっており、少し色々とあったが、最終的に zen2 ではなく raven2 を関数名に使うということで納得してくれたようだ。  

 > I'll change it to raven2 instead of zen2. Thanks.

 > 引用元: [soc/amd/picasso: Add helper functions for finding SOC type (I24b73145) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2051514)  

ただMartin Roth氏は最初、ソースとして WikiChip の Zen2 のページをあげており、  
そのページのリビジョンを見ると、2020/02/13 に Dali はZen2ベースのCPUコアではないと削除されたことが確認できる。  
[Revision history of "amd/microarchitectures/zen 2" - WikiChip](https://en.wikichip.org/w/index.php?title=amd/microarchitectures/zen_2&action=history)  
[Latest revision as of 11:22, 13 February 2020](https://en.wikichip.org/w/index.php?title=amd/microarchitectures/zen_2&diff=next&oldid=95311)  

氏がパッチを投稿したのは 2020/02/12。  
つまりコードを記述している時は WikiChip に Dali は Zen2ベースだよ、とあった訳で、一概に攻める気にはなれない。  

さらに言えば、2020/02/13 に削除されるまで、Dali は最大で 4-Core/8-Thread （かも）とも書かれており、これも間違いだ。  
そもそもいつページに追加されたのかを辿ると、2019/12/31 となっており、その時点ではまだ正式に発表はされていない。  
未確定の情報を開発者含む多くの人から信頼されているページに載せる、編集を受け入れるのはどーなの、と思ったりするが、  
これは Wikipedia が設立当初から抱えていた問題であるし、Wiki のシステムを使っているところはどこもそうだ。  
今更あーだこーだ言ってもどうしようもない。  

前置きが少し長くなってしまったが、今回書きたいと思ったのはどれだけ Zen APU がややこしいかについてだ。  

### 前提情報
Zen CPUとVega GPUを組み合わせた **Zen APU** は既に市場にいくつも存在するが、  
それらを開発コードネームで区別すると、Raven、Picasso、Raven2、Dali、Pollock、Renoirの6種類に分けられる。  

内 Dali と Pollock はRaven2ベース（別リビジョン?）であり、基本 (Raven2 == Dali == Pollock) という認識でいい（はず）。  
Dali と Pollock の違いは、i2cコントローラーの数くらいしか（私が）わかっていないが、  
Pollock の方が多く、Pollock は組み込み向けとしての色が強いとは言える。  
[soc/amd/common: Determine # of i2c controllers at runtime (I397b074e) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2057468)  

それぞれ具体的な製品には、

 * Raven
	* Ryzen 5 2400G /2200G
	* Athlon 200GE /220GE /240GE
	* Ryzen 3 2200U /2300U
	* Ryzen 5 2500U /2600H
	* Ryzen 7 2700U /2800H
	* Ryzen Embedded V1202B /V1605B /V1756B /V1807B
 * Picasso
	* Ryzen 5 3400G /3200G
	* Ryzen 3 3300U
	* Ryzen 5 3500U /3550H /3580U
	* Ryzen 7 3700U /3750H /3780U
	* Ryzen 7 3780U Surface Edition (Winston)
	* Ryzen 5 3580U Surface Edition (Winston)
 * Raven2
 	* Ryzen 5 3200U
	* Athlon 300U
	* Athlon 3000G
	* Ryzen Embedded R1505G /R1605G 
		* Dali
			* Ryzen 3 3250U
			* Athlon Gold 3150U
			* Athlon Silver 3050U
 * Renoir
 	* Ryzen 7 4700U /4800U /4800H
	* Ryzen 5 4500U /4600U /4600H
	* Ryzen 3 4300U

があげられる。（不確定なものは抜いてある）  

それぞれの Raven に対しての違いは、  

 * Picasso が Raven のアーキテクチャはそのままに製造プロセスを GF 14nm から GF 12nm に変更。  
 * Raven2 は Raven からCPUコア数、GPUコア数、I/Oを一部削り、GPUアーキテクチャの微改良（バグ修正）を行なったもの。製造プロセスは Raven と同じ GF 14nm。  
 * Renoir はCPUアーキテクチャをZen2に、CPUコア数は増量（倍）、GPUアーキテクチャを Raven2 と同じものに、GPUコア数は Raven より減らし、製造プロセスを TSMC 7nm へ変更。  

となっている。  

複雑に絡まっているのは Raven、Picasso、Raven2 であり、Renoir は今の所そこまでではない。  

### CPU

まず、Zen APU全体、RavenファミリーをCPUで分け、リストで表すと以下になる。  

 * Raven Family
 	* Zen CPU (14nm)
		* Raven
		* Raven2
			* Dali
			* Pollock
	* Zen+ CPU (12nm)
		* Picasso
	* Zen2 CPU (7nm)
		* Renoir

これだけだとシンプルだが、内部に複雑にしている要素がある。  
x86_model だ。  

x86_model はCPU内部のデータの1つであり、主にソフトウェアが判別するのに使われる。  
そして、Linux Kernel内の温度センサーを読み取るためのコードを読むと、Zen系は以下のようになっている。  

 >		case 0x1:	/* Zen */
 >		case 0x8:	/* Zen+ */
 >		case 0x11:	/* Zen APU */
 >		case 0x18:	/* Zen+ APU */

 > 引用元: [k10temp.c\hwmon\drivers - kernel/git/torvalds/linux.git - Linux kernel source tree](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/drivers/hwmon/k10temp.c#n587)  

これが本当にそうなってたらいいのだが、Zen APU である Raven2 の x86_model までが 0x18 となっている。  

[AMD Athlon 3000G review - CPU-Z Screenshots - Guru3D.com](https://www.guru3d.com/articles_pages/amd_athlon_3000g_review,4.html)  
（Ext. Model が 18）  

このため、悲しいことに Raven2 は、CPU-Z 等のソフトでは Picasso と認識される。  
個人的に、Raven2 の影が薄い、下手したら全く認知されていない原因はこれなんじゃないかと考えている。  

さらに Zen+ と言っても、Zen から変更された部分はブースト制御といったソフトウェア部や、物理設計の改良によるキャッシュレイテンシの削減ぐらいで、  
アーキテクチャには全く変更されていないが、そこに拘るとまた面倒くさいことになる。  

### GPU
#### Device ID
GPUを判別するための Device ID は、Raven に 0x15DD、Picasso に 0x15D8 が割り振られているが、  
Raven2 には 0x15DD、0x15D8 の両方が割り振られている。  
そして Raven2 のマイナーバージョンとして Dali と Pollock があり、混沌さを加速させている。  

判別には Device ID だけでなく Revision ID も使われるため、大きな問題がある訳ではないが、  
コードの判定部分を複雑にさせてしまうし、人為的なミスも起こしやすい。  

Linux Kernel では Raven, Picasso, Raven2 に別々の内部的なリビジョンIDを割り振ることで、何とか他のコードでの判定を簡略しているが、  
[Line 130: dal_asic_id.h\include\display\amd\drm\gpu\drivers - ~agd5f/linux - Unnamed repository; edit this file 'description' to name the repository.](https://cgit.freedesktop.org/~agd5f/linux/tree/drivers/gpu/drm/amd/display/include/dal_asic_id.h?h=amd-staging-drm-next#n130)  
[Line 1336: amdgpu_device.c\amdgpu\amd\drm\gpu\drivers - ~agd5f/linux - Unnamed repository; edit this file 'description' to name the repository.](https://cgit.freedesktop.org/~agd5f/linux/tree/drivers/gpu/drm/amd/amdgpu/amdgpu_device.c?h=amd-staging-drm-next&id=b989531b1f192a77c739a2976953e241d78229a3#n1336)  

Raven2, Dali, Pollock はそうではなく、判定するための部分が長ったらしくなってしまっている。  
[Line 150: dal_asic_id.h\include\display\amd\drm\gpu\drivers - ~agd5f/linux - Unnamed repository; edit this file 'description' to name the repository.](https://cgit.freedesktop.org/~agd5f/linux/tree/drivers/gpu/drm/amd/display/include/dal_asic_id.h?h=amd-staging-drm-next#n150)  

ハードウェアの情報を追う趣味を持っている人間にもひたすらにややこしいと不評である。  

またもChromimu OSへのパッチからとなるが、VBIOS部分もDevice IDの複雑さの影響を受けており、  
こんなことをKconfigの説明部分に書かれたりしている。  

 > Even though the hardware has the same vendor/device IDs, the vBIOS  
 > contains a \*different\* device ID, confusing the situation even more.  

 > 引用元: [Rework map_oprom_vendev to add revision check and mapping (I2978a569) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2040455)  

素直に Raven, Picasso, Raven2, Dali, Pollock へ違うDevice IDを割り振れなかったのだろうか？  
Dali, Pollock は Raven2ベースであるため同一IDであることはまだ納得行くが、  
Raven2自体のDevice IDを Raven, Picasso に被せなかった方が良かったと言える。  

#### アーキテクチャ
前述したように、Raven2、Renoir では Raven、Picasso にGPU部のバグが修正されており、  
GPUアーキテクチャに割り振られる GFX ID は、Raven、Picasso が **gfx902** 、  
Raven2、Renoir が **gfx909** となっている。  

そのため、x86_model で Picasso と Raven2 を一緒くたにすることは、この違いが無視されやすくなることに繋がる。  

ただ、言ってしまうとこれも問題としては全く無い。  
あるとするなら気分の問題だ。  
バグが修正されたと言っても、性能にはこれといった影響は確認されてない。  
というより Picasso と Raven2 でGPU部の規模が違うため、性能を比較することが難しい。  
LLVMのバージョンが古い場合、Raven2、Renoir のGPUは **gfx902** として認識されたりもする。  

### 製品型番
これは初代Zen APU、Ryzen 5 2400G、Ryzen 3 2200Gが出た時から広くに言われていた。  
型番としてはRyzen 2000シリーズと読めるが、中身は 12nm Zen+ ではなく、14nm Zen だからだ。  
ただ、Zen+ で盛り込まれたキャッシュレイテンシの削減によるIPC向上は確認できるため、これも気にする必要はないと思う。  

しかしこれは現在進行系で、Ryzen 3000シリーズでAPUでないRyzenは TSMC 7nmプロセス採用と、Zen2アーキテクチャにより、省電力、性能が大きく向上したため、問題として悪化した。  
Ryzen Mobile 4000シリーズが、デスクトップ向けのRyzen 3000シリーズより進んだアーキテクチャ、Zen3となっている、なんて勘違いもたまに見かけたりする。  

## 対策
#### CPUIDを使う
CPUID は x86_model のように Picasso と Raven2 がごちゃごちゃになっておらず、  
実際にChromium OS内のコードでは CPUID が使われている。  
[Rework map_oprom_vendev to add revision check and mapping (I2978a569) · Gerrit Code Review](https://chromium-review.googlesource.com/c/chromiumos/third_party/coreboot/+/2040455/)  
（src/soc/amd/picasso/include/soc/cpu.h）  

| | CPUID |
| :-- | :---: |
| Raven | 0x00810f80 |
| Picasso | 0x00810f81 |
| Raven2 (Dali, Pollock?) | 0x00820f01 |

[coreboot/cpu.c at master · coreboot/coreboot](https://github.com/coreboot/coreboot/blob/master/src/soc/amd/picasso/cpu.c#L131)  

#### 地道に情報収集
自分がやっているのがこれ。  
[AMDGPUのDID、RID、Productのびみょうまとめ Part2 | Coelacanth's Dream](/posts/2019/12/30/did-rid-product-matome-p2/)  

ソースコード、ドライバー内のファイル、ハードウェア情報を表示するソフトのスクリーンショット等から地道に集めている。  
あるGPU、APUが何であるか判明するまで時間がかかることもあるが、一度わかってしまえば紐は解ける。  

問題点は信頼度と注目度。  
どっちも自分とこのサイトにはない。  

#### 気にしない
複雑に見るから複雑なのだ。  

製品を買うのに大半の人はこれまで書いてきたことは気にしないだろうし、公式や各種メディアから、製造プロセス、アーキテクチャ等の結論として存在するベンチマーク結果が示されているのだから、それと価格で決めれば良い。  

<br>
それなのにここまで記事を書いたのは、開発者が Wiki系サイトに頼らなくていいようAMDが資料をもっと公表し、  
多くの人の苦労が詰まった半導体がどんな設計で、どんな名前を付けられたか、出来るだけ無視されないことを願っているからだ。  
