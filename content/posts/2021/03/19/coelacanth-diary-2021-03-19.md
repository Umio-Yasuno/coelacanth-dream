---
title: "【雑記】 VanGogh APU のメモリインターフェイスは (恐らく) LPDDR5 64-bit"
date: 2021-03-19T00:49:21+09:00
draft: false
tags: [ "VanGogh", ]
# keywords: [ "", ]
categories: [ "Diary", "Hardware", "AMD", "APU" ]
noindex: false
# summary: ""
---

僕はジャックの苦悩を生み続ける親切心です。  

## Source code {#src}

前回 *VanGogh APU* のブートログを元に記事を書いたが、その補足的な雑記。  
{{< link >}} [AMD VanGogh APU ブートログ | Coelacanth's Dream](/posts/2021/03/17/vgh-bootlog/) {{< /link >}}

結論から、DDR5 256-bit というのは誤った情報が出力されたもので、実際には LPDDR5 64-bit と考えられる。  

 >        [   99.984978] [drm] Detected VRAM RAM=1024M, BAR=1024M
 >        [   99.984981] [drm] RAM width 256bits DDR5
 >        [   99.985223] [drm] amdgpu: 1024M of VRAM memory ready
 >        [   99.985233] [drm] amdgpu: 3072M of GTT memory ready.
 >
 > {{< quote >}} [slow boot with 7fef431be9c9 ("mm/page_alloc: place pages to tail in __free_pages_core()")](https://lists.freedesktop.org/archives/amd-gfx/2021-March/060563.html) {{< /quote >}}

まず、元々は Linux Kernel のブートログ (`dmesg`) で、Linux Kernel はオープンソースで開発、公開されているのだから、当然どこかにそのメッセージを出力するコードが記述されている。  
それがどこかと言えば [linux/amdgpu_object.c](https://github.com/torvalds/linux/blob/master/drivers/gpu/drm/amd/amdgpu/amdgpu_object.c) で、メモリ情報自体は [linux/amdgpu_atomfirmware.c](https://github.com/torvalds/linux/blob/master/drivers/gpu/drm/amd/amdgpu/amdgpu_atomfirmware.c) にてソースコードの外にあるファームウェアバイナリから読み取っている。  
[linux/amdgpu_atomfirmware.c](https://github.com/torvalds/linux/blob/master/drivers/gpu/drm/amd/amdgpu/amdgpu_atomfirmware.c) のメモリ情報を取得する部分のコードが以下。APU の場合はファームウェアにある `mem_channel_number` に 64 を掛けてメモリバス幅としている。  

 >        	if (amdgpu_atom_parse_data_header(mode_info->atom_context,
 >        					  index, &size,
 >        					  &frev, &crev, &data_offset)) {
 >        		if (adev->flags & AMD_IS_APU) {
 >        			igp_info = (union igp_info *)
 >        				(mode_info->atom_context->bios + data_offset);
 >        			switch (frev) {
 >        			case 1:
 >        				switch (crev) {
 >        				case 11:
 >        				case 12:
 >        					mem_channel_number = igp_info->v11.umachannelnumber;
 >        					if (!mem_channel_number)
 >        						mem_channel_number = 1;
 >        					/* channel width is 64 */
 >        					if (vram_width)
 >        						*vram_width = mem_channel_number * 64;
 >        					mem_type = igp_info->v11.memorytype;
 >        					if (vram_type)
 >        						*vram_type = convert_atom_mem_type_to_vram_type(adev, mem_type);
 >        					break;
 >        				default:
 >        					return -EINVAL;
 >        				}
 >        				break;
 >        			case 2:
 >
 > {{< quote >}} [linux/amdgpu_atomfirmware.c at 5ee88057889bbca5f5bb96031b62b3756b33e164 · torvalds/linux](https://github.com/torvalds/linux/blob/5ee88057889bbca5f5bb96031b62b3756b33e164/drivers/gpu/drm/amd/amdgpu/amdgpu_atomfirmware.c) {{< /quote >}}

そして、メモリタイプ (`vram_type`) もここで取得されるが、メモリとしてのアーキテクチャに違いがあろうと、LPDDR4 は DDR4 と、LPDDR5 は DDR5 と認識される。  

 >        	if (adev->flags & AMD_IS_APU) {
 >        		switch (atom_mem_type) {
 >        		case Ddr2MemType:
 >        		case LpDdr2MemType:
 >        			vram_type = AMDGPU_VRAM_TYPE_DDR2;
 >        			break;
 >        		case Ddr3MemType:
 >        		case LpDdr3MemType:
 >        			vram_type = AMDGPU_VRAM_TYPE_DDR3;
 >        			break;
 >        		case Ddr4MemType:
 >        		case LpDdr4MemType:
 >        			vram_type = AMDGPU_VRAM_TYPE_DDR4;
 >        			break;
 >        		case Ddr5MemType:
 >        		case LpDdr5MemType:
 >        			vram_type = AMDGPU_VRAM_TYPE_DDR5;
 >        			break;
 >        		default:
 >        			vram_type = AMDGPU_VRAM_TYPE_UNKNOWN;
 >        			break;
 >        		}
 >
 > {{< quote >}} [linux/amdgpu_atomfirmware.c at 5ee88057889bbca5f5bb96031b62b3756b33e164 · torvalds/linux](https://github.com/torvalds/linux/blob/5ee88057889bbca5f5bb96031b62b3756b33e164/drivers/gpu/drm/amd/amdgpu/amdgpu_atomfirmware.c) {{< /quote >}}

LPDDR4/LPDDR5 は内部的には 4-Channel のインターフェイスを持つ。[^lpddr4_5]  
そのため、ファームウェアが LPDDR5 のメモリチャネル数を正しく理解し、正直に 4-Channel だということを告げても、Linux Kenrel の GPUドライバーは今の所それに 64 を掛けてメモリバス幅としてしまう。  

 >       +-------------+--------+---------+
 >       | Memory Type | Max Ch | Max DPC |
 >       +-------------+--------+---------+
 >       | DDR4        |      1 |       2 |
 >       | LPDDR4(X)   |      4 |       1 |
 >       | LPDDR5      |      4 |       1 |
 >       | DDR5        |      2 |       1 |
 >       +-------------+--------+---------+
 >
 > {{< quote>}} <https://review.coreboot.org/c/coreboot/+/45192/7/src/soc/intel/alderlake/meminit.c#109> {{< /quote >}}

[^lpddr4_5]: [LPDRAM for Mobile and Embedded Applications - flyer_lpdram_mobile_embedded.pdf](https://media-www.micron.com/-/media/client/global/documents/products/product-flyer/flyer_lpdram_mobile_embedded.pdf?rev=2c45263239f84c7c84981870a06bb8b2)

以上のことと合わせ、前回も書いたが、ブートログから搭載されているメモリサイズは 7200304K {{< comple >}} 約 8 GB/7.45 GiB、予約分等があるため若干少なく表示される {{< /comple >}} と読み取れるため、LPDDR5 64-bit というのが正しいメモリ構成だと考えられる。  
それならば `[drm] RAM width 256bits DDR5` と出力されたことにも、約 8GB というメモリサイズにも説明が付く。  
DDR5 128-bit (2x2ch) という可能性もあるが、*VanGogh* のディスプレイ部関連のソースファイルに、DDR4 と LPDDR5 用のパラメーターはあるのに対し、DDR5 は無いため、LPDDR5 の可能性のが高いように思う。[^vg_clk_mgr]  
ただこれが最大構成ではなく最小構成で、それ以上のメモリインターフェイスを持っている可能性もあるため、タイトルに (恐らく) を付けている。  
それでも、ブートログから読み取れるのは DDR5 128-bit か LPDDR5 64-bit というメモリ構成である。  

[^vg_clk_mgr]: [linux/vg_clk_mgr.c at 7e60e389053e59c2efc4a9a0f2da3b2c528a0d19 · torvalds/linux](https://github.com/torvalds/linux/blob/7e60e389053e59c2efc4a9a0f2da3b2c528a0d19/drivers/gpu/drm/amd/display/dc/clk_mgr/dcn301/vg_clk_mgr.c)

 >        [    0.044181] Memory: 6858688K/7200304K available (14345K kernel code, 9659K rwdata, 4980K rodata, 2484K init, 12292K bss, 341360K reserved, 0K cma-reserved)
 >
 > {{< quote >}} [slow boot with 7fef431be9c9 ("mm/page_alloc: place pages to tail in __free_pages_core()")](https://lists.freedesktop.org/archives/amd-gfx/2021-March/060563.html) {{< /quote >}}

具体的にどことか誰とかはもはや広がり過ぎてキリがない書かないが、これで間違った情報を訂正されると嬉しく思う。  
だがほとんどはされないだろう、とも思う。  
何故なら、今や重要なのは少ない文字数と画像でどれだけリプライ、リツイート、いいねを稼ぎ、興味を引くよう抜き出し、誇張したタイトルでいくらリンクをクリックさせ、広告を見せるかだからだ。  
インターネット上でそうでない場所を探すのは難しくなってきた。  
以前 Google広告を無くした時には書かなかったが、無くした理由にはそういったものに嫌気が差していた、というのもある。読み込まれるリソース量をコントロールするのが難しいのも 1つ。  
それに世界的に見ればマイナーな日本語で書いてるから、読む人も少ないだろう。  

Linux Kernel はオープンソースであり、自分が普段記事のネタにしてるのも OSS へのパッチであり、インターネットに接続されていれば誰でもソースを確認できる。  
そういう点で、自分は Leaker とは無縁で、パッチとコードを眺めるだけの存在だ。  

{{< ref >}}
 * [LPDRAM for Mobile and Embedded Applications - flyer_lpdram_mobile_embedded.pdf](https://media-www.micron.com/-/media/client/global/documents/products/product-flyer/flyer_lpdram_mobile_embedded.pdf?rev=2c45263239f84c7c84981870a06bb8b2)
{{< /ref >}}
