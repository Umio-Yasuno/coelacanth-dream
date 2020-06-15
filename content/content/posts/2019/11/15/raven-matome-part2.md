---
title: "Raven Matome Part2"
date: 2019-11-15T15:34:23+09:00
draft: false
tags: ["Radeon", "GCN", "Raven", "Raven2", "Renoir", "Dali", "Picasso", "GFX9", "gfx902", "gfx909" ]
keywords: [ "Radeon", "Raven", "APU" ]
categories: [ "Hardware", "AMD", "APU", "GPU" ]
---

Renoir、Daliにおいて ? が付いているのは、半分くらいが推測によるものだということを示す。

|   | Raven | Raven2 | Picasso | Renoir | Dali |
| :--- | :---: | :---: | :---: | :---: | :---: |
| CMOS CPU | 14nm | 14nm | 12nm | 7nm? | 12nm? |
| CPU | Zen(+) | Zen(+) | Zen+ | Zen2 | Zen+? |
| Max CPU Core/Thread | 4/8 | 2/4 | 4/8 | ?? | 2/4? |
| L3$ CPU | 4MB | 4MB | 4MB | ?? | 4MB? |
| CMOS GPU | 14nm | 14nm | 12nm | 12nm? | 12nm? |
| GPU | Vega (GFX9) | Vega (GFX9) | Vega (GFX9) | Vega (GFX9) | Vega (GFX9) |
| GPU Clock | 1300 MHz | 1200 MHz | 1400 MHz | 
| ShaderEngine | 1 | 1 | 1 | | 1? |
| Max CUs | 11 | 3 | 11 | | 3? |
| Max TMUs | 44 | 12 | 44 | | 12? |
| Max ROPs | 8 | 4 | 8 | | 4? |
| L2$ GPU | 1MB | 0.5MB | 1MB | | 0.5MB? |
| Memory Type | DDR4 | DDR4 | DDR4 | LP/DDR4/X | DDR4 |
| Support Memory Speed | 2933 MHz | 2400 MHz | 2933 MHz | 4266 MHz | 2933 MHz? |
| VCN ver | 1.0 | 1.0 | 1.0 | 2.0 | 1.0? |
| DCN ver | 1.0 | 1.0 | 1.0 | 2.1 | 1.0 |
| DeviceID | 15DD | 15D8 /15DD | 15D8 | 1636 | 15D8 |
| GPU ID | gfx902 | gfx909 | gfx902 | gfx909 | gfx909? |
| DieSize | 209.78 mm2 | 154.68 mm2? | 209.78 mm2 | | |

DaliはとりあえずRaven2をそのまま12nmにしたものと考えている。  
RenoirのCPU部は根拠に乏しいが、やろうと思えばこういった構成も可能だろう。  

それにしてもRenoirはCPUといい、メモリといい、他のAPUを大きく突き放している。  
競合がGPU部を強化してきたのを思えば当然なのかもしれないが、その分ローエンドRadeonが割を食いそうだ。  
考えようによっては、ミドルはNavi14が、ローエンドはAPUが置き換わることでPolarisもようやく引退できるかもしれない。  
何しろそろそろ4年目に突入するのだから。  

[Part1](/posts/2019/11/06/raven-matome/)
