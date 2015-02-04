title: jspmを使ってみる
date: 2015-02-02 17:00:25
tags:
---

 * _jspm ver 0.11.2のお話です_

## [jspm](http://jspm.io/)

仕事で使うのはオールインワンなWebpackが手軽だっけど、ES6を意識したモジュール管理の学習＆慣れを目的にjspmを使い始めてみた。

## jspm をインストールする

```sh
npm install jspm -g
```

## jspm initで初期化する

コマンドは`jspm init`で`package.json`に`"jspm"`prefixが追加される（無ければ作られる）

```sh

$ jspm init
# package.jsonが無いので作りますか？
Package.json file does not exist, create it? [yes]: yes
# package.json のプレフィックスにjspmを含めますか？
Would you like jspm to prefix the jspm package.json properties under jspm? [yes]: yes 
# プロジェクト名は？
Enter a name for the project (optional): gulpjspm
# baseURLとして使うフォルダ名は？
Enter baseURL (public folder) path [.]: client
# プロジェクトのコードをおさめるフォルダは？
# このフォルダにある物は.gitignoreに入れても良い、jspm installで復元される
Enter project code folder [client/lib]: client/app   
# jspmでパッケージ管理されるフォルダは？
Enter jspm packages folder [client/jspm_packages]: 
# 設定ファイルを作りますか？
Enter config file path [client/config.js]: 
Configuration file client/config.js doesn't exist, create it? [yes]: 
ok   The gulpjspm/* path has been set to app/*.js.
     To alter this path, set the directories.lib in the package.json or run jspm init -p to set the code folder.

# ES6コンパイラはどちらを使いますか? Traceur または 6to5
Which ES6 transpiler would you like to use, Traceur or 6to5? [traceur]:     
ok   Verified package.json at package.json
     Verified config file at client/config.js
     Looking up loader files...
       traceur-runtime.js

```

## チュートリアル

個人的にチュートリアルでおおっと思った部分を。

[jspm構築マニュアル](https://github.com/jspm/jspm-cli/wiki/Getting-Started)

```sh
jspm install npm:lodash-node
jspm install github:components/jquery
jspm install jquery
jspm install myname=npm:underscore
```

の結果は以下のようなディレクトリ構成になる
ぱっと見た感じ`jspm install パッケージサブフォルダ:ライブラリ`という形っぽい

```sh
├── config.js
├── jspm_packages
│   ├── es6-module-loader.js
│   ├── es6-module-loader.js.map
│   ├── es6-module-loader.src.js
│   ├── github
│   │   ├── components
│   │   │   ├── jquery@2.1.3
│   │   │   └── jquery@2.1.3.js
│   │   └── jspm
│   │       ├── nodelibs-process@0.1.0
│   │       └── nodelibs-process@0.1.0.js
│   ├── npm
│   │   ├── lodash-node@3.0.1
│   │   ├── lodash-node@3.0.1.js
│   │   ├── process@0.10.0
│   │   ├── process@0.10.0.js
│   │   ├── underscore@1.7.0
│   │   └── underscore@1.7.0.js
│   ├── system.js
│   ├── system.js.map
│   ├── system.src.js
│   ├── traceur-runtime.js
│   ├── traceur-runtime.js.map
│   ├── traceur-runtime.src.js
│   ├── traceur.js
│   ├── traceur.js.map
│   └── traceur.src.js
└── package.json

```

`jspm_packages`フォルダにインストールされる、`bower_component`っぽい


## パッケージ管理の仕組み

 + [Workflows](https://github.com/jspm/jspm-cli/wiki/Production-Workflows)
 + [jspm installマニュアル](https://github.com/jspm/jspm-cli/wiki/Installing)

よく使うコマンドをとりあえずメモる。

### 結合

以下のコマンドで関連するモジュールを結合する

```sh
jspm bundle app/main build.js
```

### install

pconfig.jsに登録する

```
jspm install パッケージ名
```

### inject

JSライブラリを`jspm_packages`にダウンロードしつつ、`config.js`に設定する
下で紹介する`jspm setmode`を使う事で、リモート(CDN)/ローカルインストール参照を切り替えれる。

```sh
 jspm inject パッケージ
```

### CDNの切り替え(リモート or ローカル)

以下のコマンドで、リモート(githubリポジトリとか)参照するのか、ローカルの`jspm_packages`を参照するのかを切り替える事が出来る。

CDNを参照する設定

```sh
jspm setmode remote
```

`jspm_packages`ローカルファイルを使う設定

```sh
jspm setmode local
```

### インストール、注入されたパッケージの更新確認

jspmのパッケージを必要に応じて最新にする

```sh
jspm update
```

## とりあえず

パッケージ管理や最低限の開発タスクが入ったモダンな`bower`って感じがする。

ちょっとプロトタイプで使ってみた感じ、モジュール管理や開発フロー全般がすごいシンプルになったのと、個人的に考えていたES6を意識したアプリの構築にぴったりで、思った以上に好きになれた。

コンパクトで取り回しが良く、個人的にはbowerやWebpackよりも好きになれそう。

調べているなかで、なるほどと思えた記事も紹介「[JSPMとWebpackの比較](https://gist.github.com/OliverJAsh/bcc676e381a06dbb3be0)」参考までに

あと参考になった[日本語の記事](http://qiita.com/hrsh7th@github/items/0a225c46ba17196b9a55)

どっかで　jspmリファレンス翻訳＆査読会でもひらきたいな。
