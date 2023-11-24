---
title: "GPU が Xe-LPG Plus となり DPAS 命令をサポートする Intel Arrow Lake"
date: 2023-11-24T18:47:54+09:00
draft: false
categories: [ "Intel", "GPU" ]
tags: [ "Arrow_Lake", "Xe-LPG" ]
noindex: false
# summary: ""
# keywords: [ "", ]
# author: ""
---

Intel® Graphics Compiler (IGC) にて *Arrow Lake* のサポートに向けたパッチ、コミットが公開され始めた。  
その中で、*Arrow Lake* GPU は *Meteor Lake* GPU の *Xe-LPG アーキテクチャ* をベースに、いくつかの変更を加えた *Xe-LPG Plus アーキテクチャ* を採用することが示されている。  

 * [Add ARL functionality · intel/intel-graphics-compiler@27c8082](https://github.com/intel/intel-graphics-compiler/commit/27c8082e85e12df394244e3350caa83f76d04e54)

 >          #define GFX_GMD_ARCH_12_RELEASE_XE_LP_MD                 (70)
 >          #define GFX_GMD_ARCH_12_RELEASE_XE_LP_LG                 (71)
 >         +#define GFX_GMD_ARCH_12_RELEASE_XE_LPG_PLUS_1274         (74)
 >          
 >          #define GFX_GET_GMD_RELEASE_VERSION_RENDER(p)             ((p).sRenderBlockID.GmdID.GMDRelease)
 >          #define GFX_GET_GMD_RELEASE_VERSION_DISPLAY(p)            ((p).sDisplayBlockID.GmdID.GMDRelease)
 >         @@ -719,6 +720,9 @@ typedef enum __NATIVEGTTYPE
 >          #define DEV_ID_56C1                             0x56C1
 >          #define DEV_ID_56CF                             0x56CF
 >          
 >         +// ARL
 >         +#define DEV_ID_7D67                             0x7D67
 >         +
 >          #define GFX_IS_DG2_G11_CONFIG(d) ( ( d == DEV_ID_56A5 )             ||   \
 >                                           ( d == DEV_ID_56A6 )             ||   \
 >                                           ( d == DEV_ID_5693 )             ||   \
 >         @@ -752,4 +756,6 @@ typedef enum __NATIVEGTTYPE
 >                                                ( d == DEV_ID_56B2 )                              ||   \
 >                                                ( d == DEV_ID_56B3 ))
 >          
 >         +#define GFX_IS_ARL_S(d)  ( ( d == DEV_ID_7D67 ) )
 >         +
 >
 > {{< quote >}} [Add ARL functionality · intel/intel-graphics-compiler@27c8082](https://github.com/intel/intel-graphics-compiler/commit/27c8082e85e12df394244e3350caa83f76d04e54) {{< /quote >}}

## Xe-LPG Plus

IGC へのパッチから、*Arrow Lake* GPU の EU 構成は *Meteor Lake* GPU と基本同じであり、機能面でも *Meteor Lake* GPU がサポートする機能はカバーしている。その上でいくつかの機能追加、改良がされているように見える。  

 >          // 1. Support both sources as ACC for FP MUL
 >          // 2. Support Src2 as ACC for FP MAD
 >          bool relaxedACCRestrictions3() const {
 >         -  return false;
 >         +  return ((getPlatform() == Xe_ARL || getPlatform() >= Xe2) &&
 >         +          !getOption(vISA_disableSrc2AccSub));
 >          }
 >         
 >
 > {{< quote >}} <https://github.com/intel/intel-graphics-compiler/commit/2998e867d91dec7198b77aa589e82e65c26ad45f#diff-0da9c3c13667e970b792d793469b3cae8569c08353c89c56c7fab4cc1a92cb60> {{< /quote >}}
 >
 >         +bool has64bundleSize2GRFPerBank() const { return getPlatform() == Xe_ARL; }
 >
 > {{< quote >}} <https://github.com/intel/intel-graphics-compiler/commit/2998e867d91dec7198b77aa589e82e65c26ad45f#diff-0da9c3c13667e970b792d793469b3cae8569c08353c89c56c7fab4cc1a92cb60> {{< /quote >}}

そして、*Arrow Lake* GPU は *Meteor Lake* GPU と異なり、XMX (Xe Matrix eXtension) ユニットを搭載し、行列積和演算命令、`DPAS (Dot Product Accumulate Systolic)` をサポートすると思われる。  
`hasDPAS()` 内の判定に `Xe_ARL` を除外するような変更はされておらず、また `Xe_ARL` における `DPAS` 命令のレイテンシ情報が追加されているからだ。  

*Meteor Lake* GPU が XMX ユニットを搭載しなかったことについては、ダイサイズの削減や、XeSS (Xe Super Sampling) は XMX ユニットが無くても実行可能であることが理由として考えられる。  
しかし、*Arrow Lake* GPU では XMX ユニットを搭載する判断をしたようだ。  
*Arrow Lake* の Graphics Tile の製造プロセスはまだ不明だが、製造プロセスの変更等により搭載してもダイサイズがそれほど問題にならなくなったのか、XeSS を用いたゲームにおける性能を重視したのかもしれない。  

 >         bool hasDPAS() const {
 >           return getPlatform() >= Xe_XeHPSDV && getPlatform() != Xe_MTL;
 >         }
 >
 > {{< quote >}} [intel-graphics-compiler/visa/HWCaps.inc at 2998e867d91dec7198b77aa589e82e65c26ad45f · intel/intel-graphics-compiler](https://github.com/intel/intel-graphics-compiler/blob/2998e867d91dec7198b77aa589e82e65c26ad45f/visa/HWCaps.inc) {{< /quote >}}
 >
 >         @@ -288,10 +290,36 @@ LatencyTableXe<PlatformGen::XE>::getDPASLatency(uint8_t repeatCount) const {
 >              default:
 >                return 32;
 >              }
 >         +  case Xe_ARL:
 >         +    switch (repeatCount) {
 >         +    case 1:
 >         +      return 21;
 >         +    case 2:
 >         +      return 22;
 >         +    case 8: {
 >         +      if (m_builder.has4DeepSystolic()) {
 >         +        return 32;
 >         +      }
 >         +      return 46;
 >         +    }
 >         +    default:
 >         +      return 22; // Conservative cycle
 >         +    }
 >
 > {{< quote >}} [intel-graphics-compiler/visa/LocalScheduler/LatencyTable.cpp at 2998e867d91dec7198b77aa589e82e65c26ad45f · intel/intel-graphics-compiler](https://github.com/intel/intel-graphics-compiler/blob/2998e867d91dec7198b77aa589e82e65c26ad45f/visa/LocalScheduler/LatencyTable.cpp) {{< /quote >}}
