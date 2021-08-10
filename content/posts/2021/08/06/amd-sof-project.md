---
title: "AMD が Sound Open Firmware プロジェクトに参加か"
date: 2021-08-06T01:09:22+09:00
draft: false
tags: [ "Renoir", "Cezanne", "VanGogh" ]
# keywords: [ "", ]
categories: [ "Software", "AMD", "APU" ]
noindex: false
# summary: ""
---

AMD のソフトウェアエンジニアであり、オーディオモジュールを担当している [Bala Kishore Pati](https://in.linkedin.com/in/bala-kishore-pati-578a7aa6/) 氏により、  
*Renoir APU* に搭載されている ACP (Audio CoProcessor) のサポートを、SOF (Sound Open Firmware) に追加するプルリクエストが投稿されている。  

 * [Enabling SOF Support For AMD Platform by balakishorepati · Pull Request #4556 · thesofproject/sof](https://github.com/thesofproject/sof/pull/4556)
  * [Enabling SOF Kernel support for AMD Platform by ajitkupandey · Pull Request #3054 · thesofproject/linux](https://github.com/thesofproject/linux/pull/3054)

SOF はプラットフォームとアーキテクチャに依存せずに、オーディオDSP のファームウェアと開発ツールを提供することを目的としたオープンソース・プロジェクトであり、Intel、Google といった巨大な組織がサポートを進めている。  
今回 *Renoir APU* のサポートが追加されることは AMD が SOFプロジェクトに参加することを意味し、ACP のさらなる活用が期待される。  
AMD は *Stoney Ridge* や *Picasso* 、*Raven2 (Dali/Pollock)* をベースにした Chromebook 向けプロセッサをリリースしており、今後に向けて Chrome OS におけるサポートを強化する狙いもあるのではないかと思われる。  
{{< link >}} [AMD、Chromebook向け "Zen" プロセッサを正式発表 | Coelacanth's Dream](/posts/2020/09/22/amd-chromebook-ryzen/) {{< /link >}}


## ACP (Audio CoProcessor) {#acp}

ACP について少し触れると、*Carrizo* 、*Stoney Ridge* 、*Raven* とそれ以降の AMD APU に搭載されているオーディオDSP であり、バージョンは *Carrizo/ Stoney Ridge* が *ACP 2.x* 、*Raven/ Picasso/ Raven2/ Renoir/ Lucienne/ Cezanne/ (Barcelo?)* が *ACP 3.x* 、そして *VanGogh* が *ACP 5.x* を採用しているとされる。にしても Vega APU を列挙しようとすると長くなる。  

AMD が開発している ACP の ALSAドライバーのコードを見ると、スペックらしきものが記述されているが、同バージョンでも対応する機能、フォーマット、チャンネル数、サンプリングレートが異なり、世代が同じだからスペックが同じとか、バージョンが進んでいるから単純に強化されているという訳でもないようだ。  
例えばキャプチャ機能だけを見ると、*Raven* ではサンプリングレートが 8-48 kHZ の範囲で対応していたのが *Renoir* では 48 kHz で固定となっていたり、*Renoir* で `SNDRV_PCM_INFO_BATCH (double buffering)` や一部フォーマットの対応が外されたり {{< comple >}} このあたりは最適化とも捉えられるが {{< /comple >}} している。  

 > ### Raven (ACP 3.x)
 > 		static const struct snd_pcm_hardware acp3x_pcm_hardware_capture = {
 > 			.info = SNDRV_PCM_INFO_INTERLEAVED |
 > 				SNDRV_PCM_INFO_BLOCK_TRANSFER |
 > 				SNDRV_PCM_INFO_BATCH |
 > 				SNDRV_PCM_INFO_MMAP | SNDRV_PCM_INFO_MMAP_VALID |
 > 				SNDRV_PCM_INFO_PAUSE | SNDRV_PCM_INFO_RESUME,
 > 			.formats = SNDRV_PCM_FMTBIT_S16_LE | SNDRV_PCM_FMTBIT_S8 |
 > 				   SNDRV_PCM_FMTBIT_U8 | SNDRV_PCM_FMTBIT_S32_LE,
 > 			.channels_min = 2,
 > 			.channels_max = 2,
 > 			.rates = SNDRV_PCM_RATE_8000_48000,
 > 			.rate_min = 8000,
 > 			.rate_max = 48000,
 > 			.buffer_bytes_max = CAPTURE_MAX_NUM_PERIODS * CAPTURE_MAX_PERIOD_SIZE,
 > 			.period_bytes_min = CAPTURE_MIN_PERIOD_SIZE,
 > 			.period_bytes_max = CAPTURE_MAX_PERIOD_SIZE,
 > 			.periods_min = CAPTURE_MIN_NUM_PERIODS,
 > 			.periods_max = CAPTURE_MAX_NUM_PERIODS,
 > 		};
 > {{< quote >}} [acp3x-pcm-dma.c « raven « amd « soc « sound - kernel/git/next/linux-next.git - The linux-next integration testing tree](https://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git/tree/sound/soc/amd/raven/acp3x-pcm-dma.c?h=next-20210804&id=cab396d8b22c13b424d9ba66f626f036f802658c) {{< /quote >}}

 > ### Renoir (ACP 3.x)
 > 		 static const struct snd_pcm_hardware acp_pdm_hardware_capture = {
 > 			.info = SNDRV_PCM_INFO_INTERLEAVED |
 > 				SNDRV_PCM_INFO_BLOCK_TRANSFER |
 > 				SNDRV_PCM_INFO_MMAP |
 > 				SNDRV_PCM_INFO_MMAP_VALID |
 > 				SNDRV_PCM_INFO_PAUSE | SNDRV_PCM_INFO_RESUME,
 > 			.formats = SNDRV_PCM_FMTBIT_S32_LE,
 > 			.channels_min = 2,
 > 			.channels_max = 2,
 > 			.rates = SNDRV_PCM_RATE_48000,
 > 			.rate_min = 48000,
 > 			.rate_max = 48000,
 > 			.buffer_bytes_max = CAPTURE_MAX_NUM_PERIODS * CAPTURE_MAX_PERIOD_SIZE,
 > 			.period_bytes_min = CAPTURE_MIN_PERIOD_SIZE,
 > 			.period_bytes_max = CAPTURE_MAX_PERIOD_SIZE,
 > 			.periods_min = CAPTURE_MIN_NUM_PERIODS,
 > 			.periods_max = CAPTURE_MAX_NUM_PERIODS,
 > 		};
 >
 > {{< quote >}} [acp3x-pdm-dma.c « renoir « amd « soc « sound - kernel/git/next/linux-next.git - The linux-next integration testing tree](https://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git/tree/sound/soc/amd/renoir/acp3x-pdm-dma.c?h=next-20210804&id=cab396d8b22c13b424d9ba66f626f036f802658c) {{< /quote >}}

*ACP 5.x* になる *VanGogh* を見ると、キャプチャ部は 8-96 kHZ のサンプリングレートに対応し、フォーマットも *Raven* と同様のものに戻っている。  

 > 		static const struct snd_pcm_hardware acp5x_pcm_hardware_capture = {
 > 			.info = SNDRV_PCM_INFO_INTERLEAVED |
 > 				SNDRV_PCM_INFO_BLOCK_TRANSFER |
 > 				SNDRV_PCM_INFO_MMAP | SNDRV_PCM_INFO_MMAP_VALID |
 > 				SNDRV_PCM_INFO_PAUSE | SNDRV_PCM_INFO_RESUME,
 > 			.formats = SNDRV_PCM_FMTBIT_S16_LE | SNDRV_PCM_FMTBIT_S8 |
 > 				   SNDRV_PCM_FMTBIT_U8 | SNDRV_PCM_FMTBIT_S32_LE,
 > 			.channels_min = 2,
 > 			.channels_max = 2,
 > 			.rates = SNDRV_PCM_RATE_8000_96000,
 > 			.rate_min = 8000,
 > 			.rate_max = 96000,
 > 			.buffer_bytes_max = CAPTURE_MAX_NUM_PERIODS * CAPTURE_MAX_PERIOD_SIZE,
 > 			.period_bytes_min = CAPTURE_MIN_PERIOD_SIZE,
 > 			.period_bytes_max = CAPTURE_MAX_PERIOD_SIZE,
 > 			.periods_min = CAPTURE_MIN_NUM_PERIODS,
 > 			.periods_max = CAPTURE_MAX_NUM_PERIODS,
 > 		};
 >
 > {{< quote >}} [acp5x-pcm-dma.c « vangogh « amd « soc « sound - kernel/git/next/linux-next.git - The linux-next integration testing tree](https://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git/tree/sound/soc/amd/vangogh/acp5x-pcm-dma.c?h=next-20210804&id=cab396d8b22c13b424d9ba66f626f036f802658c) {{< /quote >}}

サンプリングレートの対応範囲が広がったことは、バージョンが進んだと言える、*らしい* 部分ではあるが、他の関連するコードを見るに *VanGogh* の ACP には `DeviceID: 0x15E2` が割り当てられているらしく、これは Vega APU と同じ ID。  
こういった点も含めて、各 APU に搭載された ACP の相違点やバージョンについて分かりにくいこともあり、ACP はあまり自分には語れない部分ではある。  

 > 		#define ACP_DEVICE_ID 0x15E2
 >
 > {{< quote >}} [acp5x.h « vangogh « amd « soc « sound - kernel/git/next/linux-next.git - The linux-next integration testing tree](https://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git/tree/sound/soc/amd/vangogh/acp5x.h?h=next-20210804&id=cab396d8b22c13b424d9ba66f626f036f802658c) {{< /quote >}}

それと、上記 SOF へのプルリクエストではドライバー部が `DeviceID: 0x15E2` を対象にしており、*Renoir* プラットフォームというより Vega APU に搭載された ACP をサポートしているのではないかと思われる。[^pci-id]
今後変更されるかもしれないが、*VanGogh* の ACP も `DeviceID: 0x15E2` が割り当てられるのであれば、*ACP 5.x* のサポートも含んでいることとなる。  
[^pci-id]: [Enabling SOF Kernel support for AMD Platform by ajitkupandey · Pull Request #3054 · thesofproject/linux](https://github.com/thesofproject/linux/pull/3054/commits/1583a05743ecf504b95300ac987bbbe69db5ce84)

{{< ref >}}
 * [The Linux FoundationがSound Open Firmwareプロジェクトを歓迎 - The Linux Foundation](https://www.linuxfoundation.jp/press-release/2018/03/the-linux-foundation-welcomes-sound-open-firmware-project)
{{< /ref >}}
