title: hexoを使ってみる
date: 2014-12-08 06:33:16
tags:
---

## [HEXO](http://hexo.io/)

ライトなWebメモ的なのを探していたら、面白そうな物を見つけたので使ってみる。


{% img center /myfreebook/2014/12/08/use-hexo/002.png 500 %}


NodeJSベースのBlog生成ツールのようで、[jekyll](http://jekyllrb.com/)のような物らしい。

ちょっと前まではjekyllでやっていたが、最近NodeJSとJavaScriptにハマっているのでこちらを試してみようと思う。

### とりあえずインストール

```sh
npm install hexo -g
```

NodeJSとGitが入っていればこれだけでインストールできる。

### とりあえずページ作成

最初にやる事はこれ

```sh
hexo init hogeBlog
```
上のコマンドを実行する事で、ブログの雛形が出来る。

あとは、出来上がったディレクトリに移動してnpmモジュールをインストール

```sh
cd hogeBlog

npm install

```

次のコマンドを叩くとローカルサーバーが立ち上がる。

```sh
hexo server
```

その後、``http://localhost:4000/`` とかにアクセスすると最初に出来上がっている雛形ページが見えるようになる。

{% img center /myfreebook/2014/12/08/use-hexo/001.png 500 %}


### 画像の追加できるように設定する

``post_asset_folder: false``となっているので``post_asset_folder: true``に設定する、これで次から作れる新規投稿毎にリソースを管理できるようになる。

画像を指定する場合は以下のような専用のタグを使うと指定すると画像が見えるようになる、

```
{％ img center /myfreebook/2014/12/08/use-hexo/002.png 500 ％}

```

別段マークダウン指定でも問題ない。

```
![image](/myfreebook/2014/12/08/use-hexo/002.png)
```


### リポジトリ作成

github pagesでも公開できるようなので、リポジトリを作り``_config.yml``の``deploy:``に自分のgithubの設定を加える。

詳しくは[hexoの設定ページ参照](http://hexo.io/docs/deployment.html)

### ページ公開

ここまでの設定が巧く言っていれば、deployコマンドで公開できる。

```
hexo deploy
```
{% img center /myfreebook/2014/12/08/use-hexo/003.png 500 %}

### 最後に

画像の登録が多少面倒な事をのぞけば、普通に使いやすい。

作業記録兼、メモ帳にしつつ、テンプレート部分を色々弄って行きたい。

個人的にはマークダウンでドキュメントを書けるというのがとても楽だというだけなので、はてぶだろうがキータだろうがjekyllだろうがなんでもいいのだけれども

ヘッダ部分の機能が完全に死んでる(RSSとかサイト内部検索とか)ので、ここをまずどうにかしないと。
