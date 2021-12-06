---
title: "Raven2に新たなリビジョン、Pollock現る"
date: 2020-01-09T14:15:41+09:00
draft: false
tags: [ "Radeon", "GCN", "Raven2", "Dali", "Pollock", "GFX9", "gfx909" ]
keywords: [ "Raven2", "Pollock", "APU", "AMD" ]
categories: [ "Hardware", "AMD", "APU", "GPU" ]
---

Linux Kernel（amd-gfx）へのパッチから新たなASIC、Pollockが判明した。  
[[PATCH] drm/amd/display: add Pollock IDs, fix Pollock & Dali clk mgr construct](https://lists.freedesktop.org/archives/amd-gfx/2020-January/044548.html)  

DIDは15D8とPicasso、Raven2、Daliと共用であり、RIDは94、95、E9、EA、EBが割り当てられている。  
15DDにRavenとRaven2が、15D8にPicassoとRaven2が共用していた時点でそれなりにややこしかったのだが、これでさらにややこしくなってしまった。  
パッチのPollockの判定部分を見ても中々に面倒くさいことになっている。  

 > +#define ASICREV_IS_POLLOCK(eChipRev) (eChipRev == RAVEN2_15D8_REV_94 \
+		|| eChipRev == RAVEN2_15D8_REV_95 \
+			|| eChipRev == RAVEN2_15D8_REV_E9 \
+				|| eChipRev == RAVEN2_15D8_REV_EA \
+					|| eChipRev == RAVEN2_15D8_REV_EB)

DIDだけで判定できるようにしないのは何か特殊な事情でもあるのだろうか。  

Raven2とDali/Pollockの違いは、後者だとディスプレイコントローラー部の電圧のレベルが1つのみで、最低電圧で固定されるというものだと思われる。（読み間違えていたら申し訳ない。）  
[drm/amd/display: Implement voltage limitation for dali - agd5f/linux](https://cgit.freedesktop.org/~agd5f/linux/commit/drivers/gpu/drm/amd?h=amd-staging-drm-next&id=e42a34dec68950ebad7904f235ed2dfff5bb27b5)  
推測するに常時稼働させるシステム向けだろうか。  
ただ今ある情報だけではDaliに割り当てるRIDを追加させず、新たにPollockと付けた理由がわからない。  

APU系統では他にDID:15D9も確認されており、これにPollockが追加で割り当てられるのかそれともまた新しいコードネームを持ったASICとなるのかはまだ不明。  
[DeviceInfoUtils.cpp#L523](https://github.com/GPUOpen-Tools/common-src-DeviceInfo/blob/master/DeviceInfoUtils.cpp#L523)  


こちらで既に反映させてあるため、ややこしさの解消に役立てていただければ幸いだ。  
[AMDGPUのDID、RID、Productのびみょうまとめ Part2](/posts/2019/12/30/did-rid-product-matome-p2/#pollock-gfx909)  
