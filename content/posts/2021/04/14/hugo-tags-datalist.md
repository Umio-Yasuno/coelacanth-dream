---
title: "Hugo で <datalist> を活用したタグの選択補完機能を実装する"
date: 2021-04-14T17:05:23+09:00
draft: false
tags: [ "Hugo", ]
# keywords: [ "", ]
categories: [ "Software", "Diary" ]
noindex: false
# summary: ""
---

気付けばサイトのタグ数が 100個近くになり、自分でもタグから記事を辿るのが面倒くさくなってきたため、HTML5で追加された `<input type="text" autocomple="on">` `<datalist>` を活用してタグの選択補完機能を実装してみたのでそのメモ。  

該当部分のソースコードを Github Gist にも置いておく。  

 * [hugo-tag-autocomplete](https://gist.github.com/Umio-Yasuno/0a3885bcfa5233084b59dd1e649ded0a)

## HTML/テンプレート部 {#html}

`<input type="text" autocomple="on">`、`<datalist>` を使うことで、`<input>` への入力の選択肢に `<datalist>` 内の `<option>` 要素を示すと同時に、ブラウザの自動補完機能に対するヒントとして提供することができる。  

 * [\<input\>: 入力欄 (フォーム入力) 要素 - HTML: HyperText Markup Language | MDN](https://developer.mozilla.org/ja/docs/Web/HTML/Element/Input#htmlattrdefautocomplete)
 * [HTML の autocomplete 属性 - HTML: HyperText Markup Language | MDN](https://developer.mozilla.org/ja/docs/Web/HTML/Attributes/autocomplete)
 * [\<datalist\> - HTML: HyperText Markup Language | MDN](https://developer.mozilla.org/ja/docs/Web/HTML/Element/datalist)

それを Hugoテンプレートで活用しようとすると以下のようになる。対象のテンプレートファイルはタグ一覧を表示する `<theme>/layout/tags/taxonomy.html`。  

 > 		<!-- <theme>/layout/tags/taxonomy.html -->
 >      ~~~
 > 		  <div class="tag-comple">
 > 		    <input type="text" list="tags-list" size="20" id="input-tag-comple" autocomplete="on" onkeydown="jump_tag(event)">
 > 		    <datalist id="tags-list">
 > 		    {{- range .Pages.ByTitle -}}
 > 		      <option value="{{ .Title }}" data-url="{{- path.Base .RelPermalink -}}" />
 > 		    {{- end -}}
 > 		    </datalist>
 > 		    <div id="tag-comple-error"></div>
 > 		    <button type="button" class="sb jmp_b" onclick="jump_tag(event)">Enter</button>
 > 		    <button type="button" class="sb clr_b" onclick="clear_value()">Clear</button>
 > 		  </div>
 >      ~~~

`<input list="<name>">` と `<datalit id="<name>">` の \<name\> を同じものにすることでそれらを関連付けられる。  
`{{ range .Pages.ByTitle }}...{{ end }}` で自動補完に使う要素をずらっと出力できるのでテンプレート部はかなり楽。 .Pages に .ByTitle を付けることでタグタイトルを昇順にソートしている。  
`<option>` の value にタグタイトルを、拡張属性の data-url に実際のタグページに使われている URLのパス末尾を置いている。拡張属性は後述のページ偏移で使う。  
パス末尾部分は `{{ path.Base ... }}` で取り出している。(`/tags/<tag name>/ -> <tag name>`) [^path-base]  
`<option>` の終了タグは省略可能であり、ここではそのようにしている。サイトにもよるが、使われているタグすべてを出力するため、出来るだけ短くした方がいいだろう。  
下 2つの `<button>` はページ偏移と入力欄をクリアするスクリプトを実行するためのもの。  

[^path-base]: [path.Base | Hugo](https://gohugo.io/functions/path.base/)

## Javascript部 {#js}

あくまでも HTML とブラウザの機能で行えるのは選択肢の表示と自動補完だけであり、そこからページを偏移させるには Javascript で機能を実装する必要がある。  
以下がこのサイトで使っている Javascript のページ偏移のための部分を抜き出したコードだが、Javascript は最近になって勉強を始めたため、冗長だったり間違っている部分があるかもしれない。  

 >
 > 		function jump_tag(e) {
 > 		  if (e.key != `Enter` && e.type != `click`)
 > 		    return;
 > 		
 > 		  const tag_val  = document.getElementById(`input-tag-comple`).value
 > 		                    .trim().replace(/\s+/, ` `);
 > 		
 > 		  if (tag_val == ``)
 > 		    return;
 > 		
 > 		  const list     = document.getElementById(`tags-list`);
 > 		  const selector = list.querySelector(`[value="${tag_val}"]`);
 > 		
 > 		
 > 		  if(!selector) {
 > 		    const err_msg = document.getElementById(`tag-comple-error`);
 > 		
 > 		    err_msg.innerHTML = `"${tag_val}" not found`;
 > 		    err_msg.classList.add(`toast-err`);
 > 		
 > 		    setTimeout( () => {
 > 		      err_msg.classList.remove(`toast-err`);
 > 		    }, 2000);
 > 		
 > 		    return;
 > 		  }
 > 		
 > 		  const url      = `/tags/${selector.dataset.url}/`;
 > 		
 > 		  fetch(url, { method:         `HEAD` ,
 > 		               referrerPolicy: `no-referrer` }
 > 		    ).then( (response) => {
 > 		      if (response.ok)
 > 		        location.href = url;
 > 		    });
 > 		}
 > 		
 > 		function clear_value() {
 > 		  return document.getElementById(`input-tag-comple`).value = ``;
 > 		}

それでも自分が書いたコードを解説すると、まず最初の部分で Enterキー、ボタンクリック以外のイベントを早期リターンで弾いている。  
変数 tags_val に `document.getElementById('input-tag-comple').value` で入力欄の値を取得、その後の `.trim()` で主要な文字列前後の空白を削除、 `.replace(...)` で文字間の複数の空白を 1つの空白に置き換えている。  
`document.getElementById('tags-list')` で `datalit#tags-list` のオブジェクトを取得し変数 list に代入、`list.querySelector('[value="' + tags_val + '"]\')` で value に入力された値と一致する `<option>` 要素を取得している。  
そして変数 url に、拡張属性 data-url に入れたタグページのパス末尾を用いて作った URL を入れる。末尾の "/" は、無いと `/tags/<tag name>` のリダイレクトで `/tags/<tag name>/` を読み込む形になってしまうため入れている。  
その後、一応 fetch API で実際に URL先が存在するかをチェックしてから `location.href` に URL を入れてページ偏移を実行している。  
下の `clear_value` は入力欄を空にするだけの関数。入力欄をクリックすれば選択肢が表示されるため、選択、ページ偏移、リセットもクリックだけで行える方が便利だと考えた。  
まあ主に使うのは自分だし、実際便利。  

機能の感想としては、今までタグ一覧ページでいちいちタグを探したり、直近の記事から特定のタグページに飛んでいたが、補完機能によってタグページまでの手間を大きく減らすことができた。  
タグ一覧で後ろの方のページに置かれているタグまでは辿り着くのに通常数ページを経由し、どこに目的のタグがあるか目を動かして探す必要があったが、それが無くなる。  
補完に用いるタグ一覧は Hugoテンプレートの機能を使えば楽に生成できるため、実装に掛かるコストも小さい。  
欠点をあげるならば、`<datalist>` は HTMLファイルに直接出力されるため、ファイルサイズはどうしたって増加する。このサイトの場合、6KB 程サイズが増えた。  
だがこれも経由するページ数が減ったと考えれば大した問題にならない。  
それと、機能自体の問題ではないが、新しいAndroid版Firefox では何故かまだこの `<datalist>` による自動補完機能をサポートしていないことはとても残念。[^fenix-datalist]以前のAndroid版Firefox やパソコン向け Firefox、Microsoft Edge や Chomium系では期待通り動作してくれる。  

[^fenix-datalist]: [[Bug] datalist not supported · Issue #11088 · mozilla-mobile/fenix](https://github.com/mozilla-mobile/fenix/issues/11088)

どのように動作するかはこのサイトのタグ一覧ページから試すことができる。  

 * [Tag | Coelacanth's Dream](/tags/)


{{< ref >}}
 * [\<input\>: 入力欄 (フォーム入力) 要素 - HTML: HyperText Markup Language | MDN](https://developer.mozilla.org/ja/docs/Web/HTML/Element/Input#htmlattrdefautocomplete)
 * [HTML の autocomplete 属性 - HTML: HyperText Markup Language | MDN](https://developer.mozilla.org/ja/docs/Web/HTML/Attributes/autocomplete)
 * [\<datalist\> - HTML: HyperText Markup Language | MDN](https://developer.mozilla.org/ja/docs/Web/HTML/Element/datalist)
{{< /ref >}}


