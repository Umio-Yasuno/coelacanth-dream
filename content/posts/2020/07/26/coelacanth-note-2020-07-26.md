---
title: "【雑記】シーラカンスが見た夢　―― サイト移行計画失敗、Twitter、愚痴"
date: 2020-07-26T00:45:03+09:00
draft: true
# tags: [ "", ]
keywords: [ "", ]
categories: [ "Diary", ]
noindex: true
---

雑記、愚痴とかそう語ることはない情報を消費する場。  
質問等いただけたらこうした記事で回答するかもしれません。  

{{< pindex >}}

 * [サイトのロゴを微改良](#site-logo)
 * [Alder Lake-S のベンチ結果と L2cache と](#adl-s-gpu-l2)
 * [AWS S3 への移行計画失敗](#aws-migrate-fail)
 * [Twitter 始めました](#ffff-tw)

{{< /pindex >}}

## サイトのロゴを微改良 {#site-logo}
いい加減、ページの上に表示するサイトロゴが `text-shadow` で装飾しただけのテキストというのも味気ないなと思い始め、`e` を左に傾けて背景色を追加してみた。  
言ってしまえば *Serial Experiments Lain* のアレ。  

味気ないと思ったまでの経緯を言うと少し長くて、詳しくは後述するが、まず第一の計画としてサイトを AWS に移行するものがあった。  
第二の計画が、第一の計画がうまくいったら他にもサイトを立ち上げるというもの。  
このページみたいな雑記、日記をメインのサイトに載せるのではなく、`Coelacanth's Diary` みたいなタイトルを付けた別サイトに載せたかった。  
それでどうせひどく個人的なものを掲載するなら、サイトの見た目にも好きなものを詰め込もうと考えた。  
このサイトに適用しているテーマを作る時に最初目指した、 *Serial Experiments Lain* っぽい雰囲気のテーマにリベンジしようと。  

で、ある程度作ってみたが、第一の AWS への移行計画が失敗し、第二の計画も断念した。  
せっかく作ったテーマを無駄にはしたくなかったため、メインであるこのサイトのテーマに一部組み込んだというのが経緯。  

CSS だけで実現できたのは良かった。  

## Alder Lake-S のベンチ結果と L2cache と {#adl-s-gpu-l2}
[APISAK (@TUM_APISAK)](https://twitter.com/TUM_APISAK) 氏が、*Alder Lake-S* の SiSoftware の実行結果を発見し[^tw1]、その中で `512kb L2` という文字列がそのままに共有されたりしていたけど、これは正確な情報ではない。  
{{< link >}} [Details for Computer/Device Intel Alder Lake Client Platform Alder Lake Client System (Intel AlderLake-S ADP-S DRR4 CRB) : SiSoftware Official Live Ranker](https://ranker.sisoftware.co.uk/show_system.php?q=cea598ab93aa98a193b5d2efc9bb86a0c9f4d2ba87a1d9e4c2a7c2ffcfe99aa79f&l=en) {{< /link >}}

[^tw1]: <https://twitter.com/TUM_APISAK/status/1284029355182571522>

GPU系のベンチマークであるから、*Alder Lake-S* GPU の L2cache容量を表しているように思えるが、Intel GPU は L3cache をバッファやデータキャッシュ、グラフィクス用のキャッシュ等に割り当てる方式であり[^icllp-memory-cache-doc]、`512kb L2` がどれかのキャッシュ容量を示しているにしても、それは L3cache 全体ではない。  

[^icllp-memory-cache-doc]: [Volume 7: Memory Cache - intel-gfx-prm-osrc-icllp-vol07-memory_cache_0.pdf](https://01.org/sites/default/files/documentation/intel-gfx-prm-osrc-icllp-vol07-memory_cache_0.pdf)

これは他の SiSoftware 結果を見ても明らかで、L3cache 3840KB 持つ *Tiger Lake GT2* が[^tgl-gt2-l3] `1024kb L2` と表示されていたり[^sis-tgl-gt2]、AMDGPU では `16kb L2` なんて表示されていたりする[^sis-amdgpu]。  

[^tgl-gt2-l3]: [compute-runtime/hw_info_tgllp.inl at 3029db07c3135ec5fde55de369ead2c14f9b3f9c · intel/compute-runtime](https://github.com/intel/compute-runtime/blob/3029db07c3135ec5fde55de369ead2c14f9b3f9c/opencl/source/gen12lp/hw_info_tgllp.inl#L134)
[^sis-tgl-gt2]: [Details for Result ID Intel(R) Iris(R) Xe Graphics (768SP 96C 1.3GHz, 1024kB L2, 6.3GB) (OpenCL) : SiSoftware Official Live Ranker](https://ranker.sisoftware.co.uk/show_run.php?q=c2ffcbfed8b9d8e5d4e4d5e4d4e7c1b38ebe98fd98a595b3c0fdc5&l=en)
[^sis-amdgpu]: [詳細については、 デバイス Radeon RX Vega : SiSoftware公式ランキング一覧](https://ranker.sisoftware.co.uk/show_device.php?q=c9a598caabcfaac5ab8bd981a1f792f594b2d5e8c5f4d2a09dad8be2dff991ac8af2cfe98ce9d4e4c2b18cb4&l=jp)

前者は設定された URB(Unified Return Buffer) の容量、後者は CU内の L1cache を L2cache のように表示してしまっているように思うが、詳しく確かめた訳ではない。  

*Alder Lake-S* の結果から何か言えるとしたら、256SP(32EU)、URB? 512KB というのは *Gen12 (TGL /RKL) GT1* と同じ設定ということくらい。  

何かを誰かが語ったことに対して重ねることや、積極的に訂正させることはあまりしたくないからこうしたネタは扱いづらい。  
自分が絶対的に合っているとは限らないし、当然そう主張したくもない。  

## AWS S3 への移行計劃失敗 {#aws-migrate-fail}
このサイトのホスティングを AWS S3 に移行しようとしたけど失敗、断念した話。  

事の始まりは、[Liberapay](https://liberapay.com/Umio-Yasuno/donate)で寄付、支援を得るのは無理そうなので、大人しく Googleアドセンスでも導入しようとしたこと。  
他にも考えたけれども、ウェブサイトで収益を得る手段は限られている。  
記事の有料化なんてしようものなら誰も読まないだろうし、情報として誰でもアクセスでき、共有できるものの方が価値があるように思う。  
ハードウェアのレビューとか企業から案件を受ける方法がありはするのかもしれないけど、検証環境なんて禄に持ってないし、そもそもどうやってアプローチしたらいいんだろ。  
まずは活躍してるテクニカルライターに弟子入りするところからだろうか。  

ともかく、Googleアドセンスを導入しようとしたけれど、最初の認証に使うサイトの URL にサブドメインは NG で、ルートドメインかサブドメインが例外の `www` である必要があることを知った。  
それで、ドメイン取得後 `blog.coelacanth-dream.com` で運営していたこのサイトはダメということに。  

ひとまず今すぐの導入は諦め、サイトのドメインを `www.coelacanth-dream.com` に変えて運営していこうと考えた。  
そこで新たな問題が発生し、単に Github Page のカスタムドメイン設定を変えてしまうと、これまでインターネット上に貼られた `blog.coelacanth-dream.com` のリンクは機能しなくなる。  
https化してあったため、DNS側の設定でリダイレクトすることはできず、どこかで `https://blog.coelacanth-dream.com` をサイトとは別にホスティングする必要があった。  

続いて、何なら AWS のようなクラウドサービスにサイトを移行させようと考えた。  
静的ウェブサイトホスティングのサービスが存在することは知っていたため、以前から移行計画は頭の中に存在してはいた。パフォーマンスが向上すれば嬉しいし、トラフィックのモニタリング機能も使ってみたかった。  
Github Page は 1アカウント 1サイトという制限があるが、AWS や Azure 等のクラウドサービスにそういったものはない。  
どうせリダイレクト用にサイトを別に用意するなら、メインのサイトも一緒に移行させてしまおうということだ。  

で、AWS を色々試したが、結果 AWS にはリダイレクトを設置するのみとし、メインはドメインのみを変えて引き続き Github Page のお世話になることとした。  

何が問題だったのかと言うと、これも色々あったが決定的なのは AWS のツールだったのではないかと思う。  
サイトの更新作業は Github Page のおかげもあって、Hugo でのビルドからレポジトリへのプッシュまで、単純なスクリプトで済ませられている。  
それを [AWS CLI](https://aws.amazon.com/jp/cli/) 用に少し書き換えるだけと思っていたが、ローカルのディレクトリと AWS S3 のバケットを同期させる `aws s3 sync` コマンドに問題があり、ファイルが更新されたかどうかをハッシュ値ではなく、タイムスタンプで確認する方式であったため、ビルドする度ファイルを出力する Hugo とは相性が悪く、サイト更新の度にほぼすべてのファイルを転送する羽目になった。  
更新作業に掛かる時間が Github Page 時からずっと増えたし、AWS S3 の無料枠もあっという間に突破してしまった。  
CDNサービスの CloudFront も試してみて、確かに高速ではあったけど更新が中々反映されなかったりと面倒が多かった。  

タイムスタンプでファイルの更新を確認するのは AWS CLI だけでなく、他のクラウドサービスそうであるらしく、個人的に最もまともで使いやすい静的ウェブサイトホスティングサービスは Github(Gitlab) Page という結論に至った。次点で Netlify とかだろうか。  

それで結局、AWS S3 に空のバケットを設置、リダイレクトを設定、Certificate Manager で証明書を取得、CloudFront で https 等各種を設定、Route 53 で `blog.coelacanth-dream.com` のホストゾーンを設置、そして設定した CloudFront にレコードを設定、という形に落ち着いた。  
こうして書くと長いが、それぞれに大した手間は掛からず、証明書の取得に少し時間が掛かるくらいだ。CloudFront で証明書を使いたいなら バージニア北部 のリージョンで取得しなければならない、という罠に一度引っ掛かりはしたが。  

これに掛かるコストは、ホストゾーンを 1個設置したことによる $0.50(大体&yen;50) くらいで済む。  
継続的なコストもほぼ掛からない、はず。  

AWS への移行には失敗したが、当初の目的は無事に果たされたため、めでたしめでたしと言える。  
後気にしなければならないのは Googleアドセンスの導入タイミングだろうか。  
しばらく経ってからでないと承認が下りないだろうが、それがいつになるか。  

## Twitter 始めました {#ffff-tw}
記事、情報を共有して欲しいと考える以上、さすがに更新告知用の Twitterアカウントが必要だろうと思い、開設しました。  
{{< link >}} [Coelacanth's Dreamさん (@Latimeria_Dream) / Twitter](https://twitter.com/Latimeria_Dream) {{< /link >}}

ただ個人的な思想を言えば、基本 Twitter はおぞましい場であると認識している。  
淀みであり、長く居れば居るほど抜け出すことが難しくなる。  
140文字という短い文字数制限のため、自らの意図を正確に伝えることが困難となりやすい。  

あるいは情報の共有だけを目的とするならば適切なのかもしれないが、そうして機能しやすいが故に誤った情報、誤解釈もまた広まりやすい。  
また、情報自体に価値はあっても、共有する行為自体の価値は低いように思う。先のようにリスクも存在する。  
価値ある情報をさらなるものにしたいのであれば、確かな根拠を持って自らの言葉で物語る必要があるはずだ。  
そうしなければ、情報に支配されるのみだ。  

まあ何を言っても所詮は一銭にもならない個人の理念に過ぎない。偏見と言って差し障りない。  

そういうことで Twitterアカウントも可能であれば Botで運営したい。連絡手段としてもあまり使いたくはない。  
記事に Twitterの共有リンクを追加したから、自分もそれでツイートするだけでいいかもしれない。  

そういや、Twitterアカウント作成してそう経たない内に [Navy Flounder](/tags/navy_flounder) のパッチが投稿されて、それで少し自分が発信した情報が共有されたりしたけど、  
何で英語圏のメディア {{< comple >}} メディアという言い方が正しいかどうか自信が持てない {{< /comple >}} が日本語で書かれたブログなんかを基にしてんだ。パッチに含まれた解説、補足は英語で記述されているというのに。  
自分は根拠となる部分をしつこくリンクで示すようにしているから、極論、脚注にあるリンクすべて辿れば情報もまたすべて得られる。  
だけど示した根拠は丁寧に無視される。  
まともに引用もできず、リンクを貼ることもできず、鵜呑みにするだけの所もあった。  

具体的にどことかは示さないでおく。  
自分が持っているのは古臭くて頑固なハイパーテキスト思想かもしれないけど、やっぱりそれが一番だと思う。  
それと、やっぱ Twitter を基とするのは危険だと思う。生きにくい。  
あと Twitter でよく使われる絵文字って好きになれない。  


