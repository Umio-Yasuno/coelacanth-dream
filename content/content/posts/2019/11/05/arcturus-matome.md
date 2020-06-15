---
title: "Arcturus Matome"
date: 2019-11-05T19:02:05+09:00
draft: false
tags: [ "Radeon", "GCN", "Arcturus", "GFX9", "gfx908" ]
keywords: [ "Radeon", "Arcturus" ]
categories: [ "Hardware", "GPU", "AMD" ]
---

GPGPU特化な次世代GPU、Arcturusについてのまとめ。  

AMDはRDNAの発表時、GPGPU用途ではGCNを、グラフィックやゲーミング用途にはRDNAを主軸に置くと言っていた。  
Arcturusは方向性がはっきり分かれてから初めてのGCN製品となる。  
だとしても吹っ切れすぎてる部分があるように思うが。

### 性能面
まず登場時のパッチから、最大128CU構成であることが予想される。  
[drm/amdkfd: Increase vcrat size for GPU](https://lists.freedesktop.org/archives/amd-gfx/2019-July/036848.html)  
そしてまた別のパッチから、最大 8ShaderEngine となることもわかっている。  
[drm/amdkfd: Extend CU mask to 8 SEs (v3)](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd/include/v9_structs.h?h=amd-staging-drm-next&id=5145d57ec5f5cf7dadaa6ccd9c9f1e4dae82570b)  
これまで最大64CU、4SEだったことを考えると、AMDがArcturusという名前を使ったのにも納得がいく。  
[Universe Size Comparison 3D](https://www.youtube-nocookie.com/embed/i93Z7zljQ7I?start=134)  

当然Infinity FabricでのマルチGPUに対応していると思われ、EPYC Romeに合わせてCCIXでの接続にも対応してるかもしれないがまだわからない。  

GPGPU特化ということで映像出力がない、どころか3D Engineも持ってない。  
RadeonSIの方にはグラフィック機能がなくても、マルチメディアとコンピュートには使えるようコードが追加された。（まだフラグだけだが）  
[ radeonsi: add AMD_DEBUG=nogfx for testing ](https://gitlab.freedesktop.org/mesa/mesa/commit/417ab8ef6b82c868d7df16630c9a52bb71949b7b)  
後はbfloat16に対応した。  
bfloat16にはARMも対応するため、トレンドを考えると当然かもしれない。  
[Armが次々世代CPUコア「Matterhorn」の技術を発表-PC Watch](https://pc.watch.impress.co.jp/docs/column/kaigai/1211845.html)

完全にGPGPUとして吹っ切れたアーキテクチャとなっている。  
NVIDIAの方はVoltaでGPGPU向けのアーキテクチャに振ったが、グラフィック部分はそのままで、TITAN VやQuadro GV100といった製品を出していたのに対し、  
グラフィックはRDNAで行く、という気概が感じられる。  

### その他
製品名はとりあえず MI100 だけがわかっている。  
[MI100 merge (#1951)](https://github.com/ROCmSoftwarePlatform/MIOpen/commit/bc7d47bd0d65b331667da74ba3cdb04893a77998)  
  
ArcturusにはVF[^1]用のDeviceIDがあり、
[^1]:Virual Function、<https://www.amd.com/en/graphics/workstation-virtual-graphics>

その関係かVCNを2つ搭載している。  
VCN2.0では4ストリームのエンコードに対応していたため、Arcutusでは最大8ストリームとなるはずだ。  
[「Radeon RX 5700」と「Radeon RX 5700XT」を試す - 第3世代Ryzen+NAVI徹底攻略-マイナビニュース](https://news.mynavi.jp/article/20190707-855531/7)  
一応、ArcturusではVCNのバージョンが2.5となっているが、2.0との違いは（まだ私が）わかっていない。  

