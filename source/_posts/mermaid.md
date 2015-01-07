title: hexoにmeamaidを組み込んでみるテスト
date: 2015-01-06 14:17:59
tags:
---

最近[meamaid](https://github.com/knsv/mermaid)での図形作成にはまってる。

で、hexoオリジナルテーマにmeamaidを組み込んだけれど、単純に組み込んだだけだと動作しなかったのでメモ。

#### 動かなかった原因

 + `<br>`タグとか混ざる
 + `--`が`-`に変換される。

#### javascriptから突っ込んでみるとできる。

```html
<div id="drowmmd">
<script>
  (function(){
    document.getElementById('drowmmd').innerHTML 
    = '<div class="mermaid"> graph TD; A-->B; A-->C; B-->D; C-->D; </div>'
  })();
</script>
```

で、以下のようにちゃんと変換されて表示される。

<div id="drowmmd"></div>
<script>
  (function(){
    document.getElementById('drowmmd').innerHTML = '<div class="mermaid"> graph TD; A-->B; A-->C; B-->D; C-->D; </div>'
  })();
</script>

けれども個人的には、mermaid構文のようにもう少し簡単に書きたい。

```
<div class="mermaid">
 graph TD;
   A-->B;
   A-->C;
   B-->D;
   C-->D;
</div>
```
ので、カスタムmarkdown機能を組み込んでみる。

[hexo-renderer-marked-plus](https://github.com/akfish/hexo-renderer-marked-plus) でMarkdownの内容をオーバライドできるようなので、ひとまずHexoにinstallする。

 + hexoフォルダに以下を実行

```sh
npm install hexo-renderer-marked-plus --save
```

で、`themes/自分のテーマ/scripts/markedrenderer.js`に以下のコードを追加して、独自解釈をできるように。


```javascript
var marmaid ={
  inline: /^mermaid\n/
};

hexo.markedRenderer = {
  codespan: function(code) {
    if (marmaid.inline.test(code)) {
      return '<div class="mermaid">' + code.replace(marmaid.inline,'') + '</div>'
    }
    return this._super.codespan(code);
  }
}
```

これで以下のように書くと。

```
`mermaid
 graph TD;
   A-->B;
   A-->C;
   B-->D;
   C-->D;
`
```

下のようにhexo上でmermaidが書けるようになる。

`mermaid
 graph TD;
   A-->B;
   A-->C;
   B-->D;
   C-->D;
`

mermaid自体、最近シーケンス図書けるようになったりしてるので、今のうちにくみこんでみたけれど
merkdownライクにフローチャートやシーケンスが書けるのは凄い良い気がする。

`mermaid
sequenceDiagram
    Hexo->>Mermaid: Covert?
    Hexo-->Mermaid: Drow!
`

次回調べたいもの

+ [Angular WebRTC](https://github.com/mgechev/angular-webrtc)

