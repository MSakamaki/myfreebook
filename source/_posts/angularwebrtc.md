title: Multi-User Video Conference with WebRTCの意訳
date: 2015-01-08 20:12:08
tags:
---

WebRTCの教材として凄く良さそうだったので、[Angular WebRTC](https://github.com/mgechev/angular-webrtc)について、[Blog](http://blog.mgechev.com/2014/12/26/multi-user-video-conference-webrtc-angularjs-yeoman/)のざっくり訳をしてみた。

---

これは[WebRTC](http://webrtc.org)と[AngularJS](http://angularjs.org)、[Yeoman](http://yeoman.io/)で複数人で会議するためのビデオ会議アプリを作る為のチュートリアルです。

このチュートリアルは、WebRTCのピアツーピア接続の方法と、[ICE (Interactive-Connectivity Establishment) framework](https://en.wikipedia.org/wiki/Interactive_Connectivity_Establishment)がどのように、NATトラバーサルで使用されているのかを詳細に説明します。

このプロジェクトの開発版は、我々の [Heroku](https://mgechev-webrtc.herokuapp.com)で見る事が出来ます。
ソースコードは[GitHub](https://github.com/mgechev/angular-webrtc)に置いてあります。


### なんで私はYeomanとAngularJSを選んだのか？

YeomanのGeneratorは非常に早くアプリケーション開発に必要な、すべてのボイラープレートを用意する事が出来ます。
Yeomanは本来ならば非常に面倒なものを、bashたった数行で、効率よく最適化されたアプリケーション開発環境を用意する事ができます。

```bash
grunt build
git remote add heroku git@uri
git push heroku master
```
### なぜAngularJSを使ったのか？


AngularJSは前提原則として、素敵なデータバインディングメカニズムによる粗結合を強制する仕組み、それによる明確に定義されたコンポーネント化の仕組み、そして(あなたがAngular-routerを使っていれば)out of the box形式のルーターがついてきます。


Q: AngularJSの代わりに他のものを使う事ができますか？
A: はい、あなたがその他の物を使う知識があれば大丈夫です。

難易度の高いDOM操作とビューの少ないシングルページアプリケーションの場合は、React.jsもしくはWebComponentsをお勧めします。

{% img center /myfreebook/2015/01/08/angularwebrtc/webrtc-yeoman.png 500 %}

## WebRTC イントロダクション

In my blog post ["WebRTC chat with React.js"](http://blog.mgechev.com/2014/09/03/webrtc-peer-to-peer-chat-with-react/) I already did a brief introduction about what WebRTC is and how it works:


私のブログの記事の["React.jsと、チャットWebRTC」]（http://blog.mgechev.com/2014/09/03/webrtc-peer-to-peer-chat-with-react/）で、私はすでにやったWebRTCがあり、それがどのように機能するかかについての簡単な紹介：

私のBlog記事["WebRTC chat with React.js"](http://blog.mgechev.com/2014/09/03/webrtc-peer-to-peer-chat-with-react/)で、既にWebRTCについて書いてあります。
WebRTCがどのように機能するかを簡単に紹介します。

---

RTCはリアルタイム通信(Real-Time Communication)の略です。
WebRTCが登場するまで、複数のブラウザ間通信を行う為には、そのブラウザの間にサーバーを置いてメッセージをやり取りする必要がありました。
WebRTCが実装されているブラウザは、ブラウザ間のピアツーピア通信が行えます。
NATトラバーサルフレームワークを使用し、ICEを用いてブラウザ間の最も適切なルートを見つけ、それらの通信は仲介サーバーを設ける事無く行えます。
2014年７月１日以降、WebRTCブラウザのAPI規格は[W3Cによって公開](http://dev.w3.org/2011/webrtc/editor/webrtc.html)されています。

前の記事ではチャットルームに参加するピア同士でデータチャネルを開くため、Pear.jsを使いました。

今回は標準のブラウザに搭載されているWebRTC APIを使い、WebRTCセッションの確立されている状況をもう少し深くセスメイして行きます。

あなたは深い技術的理解を目指していない場合、この章をスキップして即座にサーバの実装の章へ行く事も出来ます。

### どのように作ったWebRTCが動くのか？

以下のUMLシーケンス図を参照してください。

`mermaid
sequenceDiagram
  participant アリス
  participant Webアプリ
  participant TRUNサーバー
  participant ボブ
  %%
  アリス->>Webアプリ: 1. ボブを呼び出す
  Webアプリ->>ボブ: 2. アリスがあなたを呼んでいます。
  ボブ->>Webアプリ: 3. アリスへ回答を返します。
  Webアプリ->>アリス: 4. ボブの回答が帰ってきました。
  %%
  アリス->>アリス: 5. ICE候補を取得(ICE gathering process)
  アリス->>Webアプリ: 6. ボブにSDPを申請する
  Webアプリ->>ボブ: 7. アリスからのSDP申請が来る
  ボブ->>ボブ: 8. ICE候補を取得(ICE gathering process)
  ボブ->>Webアプリ: 9. アリスから来たSDP申請に対して回答する
  Webアプリ->>アリス: 10. ボブからのSDP申請の回答を受け取る
  %%
  alt 最適なICE候補が見つかった場合
    アリス->>ボブ: SRDP madia session
    ボブ->>アリス: SRDP madia session
  else 最適なICE候補が見つからなかった場合
    アリス->>TRUNサーバー: SRDP madia session
    TRUNサーバー->>ボブ:   SRDP madia session
    ボブ->>TRUNサーバー:   SRDP madia session
    TRUNサーバー->>アリス: SRDP madia session
  end
`
 * [SDP(Session Description Protocol)](http://tools.ietf.org/html/rfc3264)
 * [直積集合](http://ja.wikipedia.org/wiki/%E7%9B%B4%E7%A9%8D%E9%9B%86%E5%90%88)

上記のシーケンス図では、アリスが中心にあるWebアプリケーションサーバー(Webアプリ)を介して　ボブとのピア接続を確立させる方法を図解しています。

---

1. 最初にアリスはRESTfulなメソッド(`POST /call/Bob`)を呼び出す事で、アプリケーションサーバーを通してボブを呼び出します。

2. プッシュ通知を介して、アプリケーションサーバーはアリスから呼ばれている事をボブに伝えます。Webアプリはアリスの呼び出しを,WebSocketを使用して通知(notification)を送る事もあります。

3. ボブはプッシュ通知の応答に「アリスと話をしたい」と回答します。

4. Webアプリはアリスに対して、ボブの回答を返します。

5. アリスはボブが彼女の呼び出しを受け入れたので、接続の為に必要な「ICE候補の収集プロセス(ICE candidates gathering process)」開始します。次のセクションでその内容をさらに見て行きましょう。

6. アリスはICE候補のセットを持っています。(例：127.0.0.1:5545, 192.168.0.112:6642, 94.23.24.56:6655、正確には`a=candidate:1 1 UDP 2130706431 192.168.1.102 1816 typ host`、ポート：ホスト - にて、ペアとしてみる事が出来る。)アリスはICE候補といくつかの追加情報(サポートされてるビデオ/オーディオコーデック情報)が含まれたSDP申請を準備します。準備が完了すると、用意していたSDP申請を送信します。

7. Webアプリはボブへ、アリスの申請をリダイレクトします

8. ボブも自分の為のICE候補を探し始めます。

9. ボブはSDPの返信を準備(アリスのSDPオファーと似た形)し、Webアプリを経由してアリスに回答を送信します。(ボブとアリスはまだP2P接続は確立できていません。)

10. Webアプリはアリスにボブの回答をリダイレクトします。

11. アリスとボブは互いに持っているICE候補をマッチングさせ、P2P接続を確立させようとします。この段階でも、まだICE候補の収集は行われています。
 * アリスとボブは、互いに持っているICE候補の直積集合を作成します。  
ボブは自分のICE候補とアリスのICE候補同士で優先順位を組み合わせながら、それらの間の接続を確立しようとします。

もしアリスとボブが互いにICE候補を用いてのP2P接続を確立できない場合、両者は[symmetric NAT](http://think-like-a-computer.com/2011/09/19/symmetric-nat/)の後ろに居る可能性が高いです。
TURNサーバーを提供している場合、ビデオ/オーディオ接続はそのTURNサーバーを介して通信を確立できます。
それ以外の場合はP2P接続は確立できません。

#### ICE gathering process

新しい[RTCPeerConnection](http://w3c.github.io/webrtc-pc/#rtcpeerconnection-interface)を作る為には、ブラウザのWebRTC APIを利用する場合、configオブジェクトを提供します。
ここにはSTUNとTURNサーバーのセットを含めます。

```javascript
new RTCPeerConnection({ 'iceServers': [{ 'url': 'stun:stun.l.google.com:19302' }]})
```

STUNの使い方を理解する為には、まず何故それが必要になるのかの理由を知る必要があります。
まず「NAT」が何なのかを見てみましょう。

##### NAT

チュートリアルを行う前に、NATに関わる幾つかの単語の概要を説明できます。

NATはネットワークアドレス変換(Network Address Translation.)の略です。
それはパブリックIPアドレスと、内部(プライベート)IPアドレスを変換する為の非常に一般的な手法です。
パブリックIPアドレスが限られた数しかないISPプロバイダの多くは、内部ネットワークにプライベートIPアドレスを使用して数を増やし、外部に対してはパブリックIPアドレスに変換する方法を使っています。
NATとNATの異なる種類似ついての詳細は[Wiki(英語)](https://en.wikipedia.org/wiki/Network_address_translation)にあります。

特定のホスト端末がNATの背後にあるとき、それはパブリックIPアドレスを持ちません。
そのIPアドレスは[192.168.0.102のように](https://en.wikipedia.org/wiki/Private_network)なり済まします。
特定のホスト端末がネットワークにあるサービスに到達しようとする場合、ローカルネットワークの外部にはNATサーバーを介して要求を行います。
NATサーバーはNATサーバーのIPアドレスを送信元アドレスにに変換する事で、要求を自身にリダイレクトさせます。
NATはで変換されたアドレス(ローカルにある干すと端末のホスト名とポートと、NATのホスト名)に、送信元アドレス(ホスト名とポート)をマッピングするNATテーブルを作成します。
NATはリモートサービスによって応答を受信すると、要求の最初のソースを見つける為にNATテーブルを使用して、対応する応答をリダイレクトします。

##### STUN

Q： 何故STUNサーバーが必要で、それはいったい何なんですか？
A：これらの質問に答える 前に、「何故私たちはNATの背後が判らないか理解できますか？」と言う話ができます。

NATの後ろに居る私たちがリモートサービスに到達したいと仮定します。

まず、リモートサービスに要求を行った場合、リクエストの送信もとアドレスを持つサービスのレスポンスと、自分自身のマシンのアドレスを比較します。
その時にもしアドレスが異なる場合は、私たちがNATの背後に居る事が明らかです。
サービスが私たちのローカルネットワーク(s)の外に位置する必要がある事に注意してください。

Q: 私たちはどのようにすれば確実に、サービスの応答内容から受信したアドレスと、上記のNATのひとつであるかを確認できますか？
A: 私たちにはできません、ネストされたNATの例では、いくつかのNATを経由した先かもしれないが、基本的にはICEのNATトラバーサル手順は同じです。

では、リクエストの送信元アドレスを持つサービスのレスポンスで、STUNが何をするかです。

私たちのNATのIPアドレスを持っているときは、私たちはリフレクションICE候補(reflexive ICE candidate)と呼ばれる新しいICE候補を使う事が出来ます。
これは、そのネットワークアドレス変換で使用されるNATサーバーの、IPアドレスとポートの組み合わせです。


## 実装

次のステップでは、私たちのサンプルアプリケーションの実装を見て行きます。
アプリケーションは２つの主要コンポーネントでできています。

 * バックエンド - アプリケーションサーバー(シーケンス図のWebアプリ)、P2P接続が確立されるまでの間の端末間の通信を担当します。
 * Webアプリ - AngularJSアプリケーションです(シーケンス図のアリスとボブ)、実際のマルチユーザービデオチャットです。


あなたは[Heroku](https://mgechev-webrtc.herokuapp.com)を使う事でこのアプリケーションを実際に試せます。

## バックエンド

このセクションではバックエンドを実装します。
バックエンドは前にあるシーケンス図のWebアプリです。
主な基本機能は静的ファイル(HTML,JS,CSS)を提供することと、ピアによる要求のリダイレクトです。

このコンポーネントは、与えられた部屋同士で`socket.io`のピアなソケットコネクションが関連付けられており、各部屋のコネクションを維持します。

JavaScriptで私たちのWebRTCアプリケーションを実現する為に、バックエンドにはNode.jsを使用しています。

### それでは始めましょう！

```bash
mkdir webrtc-app && cd webrtc-app
# initialize the app
npm init
mkdir lib
touch index.js
```

ルートフォルダにindex.jsを作成し、以下の内容を追記します。

```javascript
var config = require('./config/config.json'),
    server = require('./lib/server');

// ポートは(Herokuの)環境変数を使用して設定されている。
config.PORT = process.env.PORT || config.PORT;

server.run(config);
```

アプリケーションの中心となる依存関係をインストールする為に、以下のコマンドを実行します。

```bash
npm install express --save
npm install socket.io --save
npm install node-uuid --save
```

では、上で作った`lib`ディレクトリに移動しましょう。


```text
cd lib
```

次に、このディレクトリ`lib`で、`server.js`と言うファイルを作ります。
内容は以下ののようになります。

```javascript
var express = require('express'),
    expressApp = express(),
    socketio = require('socket.io'),
    http = require('http'),
    server = http.createServer(expressApp),
    uuid = require('node-uuid'),
    rooms = {},
    userIds = {};

expressApp.use(express.static(__dirname + '/../public/dist/'));

exports.run = function (config) {

  server.listen(config.PORT);
  console.log('Listening on', config.PORT);
  socketio.listen(server, { log: false })
  .on('connection', function (socket) {

    var currentRoom, id;

    socket.on('init', function (data, fn) {
      currentRoom = (data || {}).room || uuid.v4();
      var room = rooms[currentRoom];
      if (!data) {
        rooms[currentRoom] = [socket];
        id = userIds[currentRoom] = 0;
        fn(currentRoom, id);
        console.log('Room created, with #', currentRoom);
      } else {
        if (!room) {
          return;
        }
        userIds[currentRoom] += 1;
        id = userIds[currentRoom];
        fn(currentRoom, id);
        room.forEach(function (s) {
          s.emit('peer.connected', { id: id });
        });
        room[id] = socket;
        console.log('Peer connected to room', currentRoom, 'with #', id);
      }
    });

    socket.on('msg', function (data) {
      var to = parseInt(data.to, 10);
      if (rooms[currentRoom] && rooms[currentRoom][to]) {
        console.log('Redirecting message to', to, 'by', data.by);
        rooms[currentRoom][to].emit('msg', data);
      } else {
        console.warn('Invalid user');
      }
    });

    socket.on('disconnect', function () {
      if (!currentRoom || !rooms[currentRoom]) {
        return;
      }
      delete rooms[currentRoom][rooms[currentRoom].indexOf(socket)];
      rooms[currentRoom].forEach(function (socket) {
        if (socket) {
          socket.emit('peer.disconnected', { id: id });
        }
      });
    });
  });
};
```

それでは上記のコードステップ毎に見て行きましょう。

```javascript
var express = require('express'),
    expressApp = express(),
    socketio = require('socket.io'),
    http = require('http'),
    server = http.createServer(expressApp),
    uuid = require('node-uuid'),
    rooms = {},
    userIds = {};

expressApp.use(express.static(__dirname + '/../public/dist/'));
```

この部分では、すべての依存関係をrequireし、作成されたexpressが静的ファイルを提供するためのディレクトリを構成/設定します。
このディレクトリは我々のアプリのルート直下にある`public`フォルダの中に位置します。

```javascript
server.listen(config.PORT);
console.log('Listening on', config.PORT);
socketio.listen(server, { log: false })
.on('connection', function (socket) {
  // 追加のロジック
});
```

socket.ioの'connection'イベント（追加のロジック部分）は、クライアントがこのサーバーに接続した事を意味します。
この接続が確立したら、対応するイベントハンドラを追加する必要があります。

```javascript
var currentRoom, id;

socket.on('init', function (data, fn) {
  // Handle init message
});

socket.on('msg', function (data) {
  // Handle message
});

socket.on('disconnect', function () {
  // Handle disconnect event
});
```

これから扱う予定の３つのイベントです。

initイベントは、ビデオチャットルームの初期化に使用します。
ビデオチャットルームが既に作成されている場合、与えられた部屋に関係したソケットのコレクションに、クライアントからのソケットを追加する事でビデオチャットルームに現在のクライアントを参加させます。(`rooms[room_id]`に、ビデオチャットルームのクライアント配列が入っています)
部屋が作成されていない場合、部屋を作成して現在のクライアントを追加します。
部屋の作成には[`node-uuid` module](https://github.com/broofa/node-uuid#getting-started)で乱数を用いて一意の部屋を作り出します。

```javascript
currentRoom = (data || {}).room || uuid.v4();
var room = rooms[currentRoom];
if (!data) {
  rooms[currentRoom] = [socket];
  id = userIds[currentRoom] = 0;
  fn(currentRoom, id);
  console.log('Room created, with #', currentRoom);
} else {
  if (!room) {
    return;
  }
  userIds[currentRoom] += 1;
  id = userIds[currentRoom];
  fn(currentRoom, id);
  room.forEach(function (s) {
    s.emit('peer.connected', { id: id });
  });
  room[id] = socket;
  console.log('Peer connected to room', currentRoom, 'with #', id);
}
```

もう一つの機能としては、クライアントが与えられた部屋に接続したときに、新しく接続されたピア接続を、同じビデオチャットルームにいる他のピア接続すべてに通知する事が必要です。

クライアントが正常に接続された後、クライアントのIDとビデオチャットルームのIDとで呼び出すコールバック関数(`fn`)を持ちます。

'msg'イベントは、別のピアから特定のピアへ、`SDP`メッセージ、もしくは`ICE`候補をリダイレクトさせます。

```javascript
var to = parseInt(data.to, 10);
if (rooms[currentRoom] && rooms[currentRoom][to]) {
  console.log('Redirecting message to', to, 'by', data.by);
  rooms[currentRoom][to].emit('msg', data);
} else {
  console.warn('Invalid user');
}
```

与えられたピアのIDには常に整数が格納されているので、
イベントハンドラの最初の行で数値として変換します。

その後、イベントデータオブジェクトの`to`プロパティで指定したピアにメッセージを送ります。

最後のイベントハンドラ(および、サーバの最後の部分)が切断を行う為のイベントハンドラです。

```javascript
if (!currentRoom || !rooms[currentRoom]) {
  return;
}
delete rooms[currentRoom][rooms[currentRoom].indexOf(socket)];
rooms[currentRoom].forEach(function (socket) {
  if (socket) {
    socket.emit('peer.disconnected', { id: id });
  }
});
```

サーバーから与えられたピアが切断された場合(例：ユーザーがブラウザを閉じるかF5等で更新をする。)、与えられた部屋のソケットリストから、該当するクライアントのソケットを削除します。(delete演算子を使う)
その後に、切断されたピアのidで、部屋の中の他のすべてのピアに対して`peer.disconnected`イベントを送ります。
このように、切断されたピアの情報が、すべての接続されたクライアントに対して送られ、クライアントのビデオ要素を削除する事が出来るようになります。

バックエンドの最後に設定を作ります。
ルートディレクトリの直下にconfigと言うディレクトリを作ります。

```bash
cd .. # 今あなたがlibディレクトリに居る場合...
mkdir config && cd config
```

新しいファイル`config.json`を作って、以下の内容を書きます。

```json
{
  "PORT": 5555
}
```

## Webクライアント

### セットアップ

yeoman generatorを使ってAngularJSのアプリケーションを作るには、次の手順に従います。

```bash
npm install -g yeoman
npm install -g generator-angular
# もしあなたがconfigディレクトリに居る場合...
cd ..
mkdir public && cd public
yo angular
```
あなたはyeomanから幾つかの質問をされますが、次のように答えましょう。

{% img center /myfreebook/2015/01/08/angularwebrtc/setup.png 500 %}

基本、依存するモジュールとして`angular-route` を必要とします。
簡単により良い見た目にする為にBootstrapを使って少しの労力で作ろうと思います。

### 実装

まず最初に、いくつかのブラウザ互換をなんとかする必要があります。
`public/app/scripts`ディレクトリの中に`adapter.js`を作り、次にその中身を書きます。

```javascript
window.RTCPeerConnection = window.RTCPeerConnection || window.webkitRTCPeerConnection || window.mozRTCPeerConnection;
window.RTCIceCandidate = window.RTCIceCandidate || window.mozRTCIceCandidate || window.webkitRTCIceCandidate;
window.RTCSessionDescription = window.RTCSessionDescription || window.mozRTCSessionDescription || window.webkitRTCSessionDescription;
window.URL = window.URL || window.mozURL || window.webkitURL;
window.navigator.getUserMedia = window.navigator.getUserMedia || window.navigator.webkitGetUserMedia || window.navigator.mozGetUserMedia;
```

FirefoxとChromeはまだmozとwebkitのベンダープレフィックスが必要なので、その部分をなんとかしましょう。

よし！これでこの未来的を感じるアプリはChromeとFirefoxの上で動作します！

これで、アプリケーション内のコンポーネントに、メディアストリームを提供する`VideoStream`と言うServiceを作る準備が整いました。

```bash
yo angular:factory VideoStream
```

で、上のサブジェネレータにより作られたファイルを編集しましょう。

```javascript
angular.module('publicApp')
  .factory('VideoStream', function ($q) {
    var stream;
    return {
      get: function () {
        if (stream) {
          return $q.when(stream);
        } else {
          var d = $q.defer();
          navigator.getUserMedia({
            video: true,
            audio: true
          }, function (s) {
            stream = s;
            d.resolve(stream);
          }, function (e) {
            d.reject(e);
          });
          return d.promise;
        }
      }
    };
  });
```

`VideoStream`は` getUserMedia`を使ってビデオストリームを提供するために`$q`を使用しています。
最初呼び出したら`getUserMedia`は、マイクとカメラを使用する権限をユーザーに訪ねます。

{% img center /myfreebook/2015/01/08/angularwebrtc/webcam-permissions.png 500 %}

ウェブカメラへのアクセス権確認が度々起こらないようにするため、ビデオストリームのアクセス権を取得した後、`stream`変数へキャッシュします。

#### アプリケーションの構成

では、これからアプリケーションのルーターを設定して行きます。
`public/app/scripts/app.js`を編集できるようにし、いかにあるようなルート定義を追加します。

```javascript
angular
  .module('publicApp', [
    'ngRoute'
  ])
  .config(function ($routeProvider) {
    $routeProvider
      .when('/room/:roomId', {
        templateUrl: 'views/room.html',
        controller: 'RoomCtrl'
      })
      .when('/room', {
        templateUrl: 'views/room.html',
        controller: 'RoomCtrl'
      })
      .otherwise({
        redirectTo: '/room'
      });
  });
```

この設定には２つのルーター設定があります。

 * `/room`
    *  初めてアプリケーションにアクセスしたユーザーの為のページです。
    1. まず、`/room`に訪れると、自分のWebカメラ(`RoomCtrl`のロジック)へのアクセスを可能にした後、自動的に新しい部屋が割り当てられ、別のURL(`/room/:roomid`)にリダイレクトされます。
    2. このURLにリダイレクトされると、自分が話したい他のユーザーにこの部屋のURLを共有する事が出来ます。

 *  `/room/:roomid` 
    * このURLを共有する事で、既に部屋を作成しているユーザーのビデオ通話に参加することができます。

わかりやすくするために、このメカニズムは単純かつ、安全ではありません、別のユーザーのセッションURLを推測することで、たいした努力もせずに別のユーザーのビデオ会話に参加できてしまいます。
他のユーザーのプライバシーを損なわないよう、礼儀正しくありましょう　:-)

では、`app.js`の下の方にconstantを追加しましょう。

```javascript
angular.module('publicApp')
  .constant('config', {
      // Change it for your app URL
      SIGNALIG_SERVER_URL: YOUR_APP_URL
  });
```

サーバとクライアントの`socket.io`接続の為にこの定数を使っています。

#### Io

それではこれから、Service`Io`を作成しましょう。

```bash
yo angular:factory Io
```

ファイル`/public/app/scripts/services/io.js`の内容に以下を書きます。

```javascript
angular.module('publicApp')
  .factory('Io', function () {
    if (typeof io === 'undefined') {
      throw new Error('Socket.io required');
    }
    return io;
  });
```

ここでは、ラッパーサービス内部でグローバルに存在する`io`をInjectできるようにしています。
この目的は、`io`テストをするときにmonkeyで簡単にモックすることができるからです。

#### RoomCtrl

Now lets create a new controller, called `RoomCtrl`:

では今から`RoomCtrl`と言う新しいコントローラを作りましょう。

```bash
yo angular:controller Room
```

ファイル`/public/app/scripts/controllers/room.js`を編集します。

```javascript
angular.module('publicApp')
  .controller('RoomCtrl', function ($sce, VideoStream, $location, $routeParams, $scope, Room) {

    if (!window.RTCPeerConnection || !navigator.getUserMedia) {
      $scope.error = 'WebRTC is not supported by your browser. You can try the app with Chrome and Firefox.';
      return;
    }

    var stream;

    VideoStream.get()
    .then(function (s) {
      stream = s;
      Room.init(stream);
      stream = URL.createObjectURL(stream);
      if (!$routeParams.roomId) {
        Room.createRoom()
        .then(function (roomId) {
          $location.path('/room/' + roomId);
        });
      } else {
        Room.joinRoom($routeParams.roomId);
      }
    }, function () {
      $scope.error = 'No audio/video permissions. Please refresh your browser and allow the audio/video capturing.';
    });
    $scope.peers = [];
    Room.on('peer.stream', function (peer) {
      console.log('Client connected, adding new stream');
      $scope.peers.push({
        id: peer.id,
        stream: URL.createObjectURL(peer.stream)
      });
    });
    Room.on('peer.disconnected', function (peer) {
      console.log('Client disconnected, removing stream');
      $scope.peers = $scope.peers.filter(function (p) {
        return p.id !== peer.id;
      });
    });

    $scope.getLocalVideo = function () {
      return $sce.trustAsResourceUrl(stream);
    };
  });
```

このコードをステップ毎に見て行きましょう。

`RoomCtrl`には次のような目的で依存関係を追加しています。
- `$sce` -video要素のソースを設定するのに使う
- `VideoStream` - ユーザーのカメラからビデオストリームを取得しキャッシュする。
- `$location` - チャットルームのURLに、ユーザーをリダイレクトするのに使います。
- `$routeParams` - ルームIDを取得するのに使います。
- `$scope` - ビューとデータのバインドを行う為に必要です。
- `Room` - これから作ろうとしているService部分、ここでP2P接続を管理します。

```javascript
if (!window.RTCPeerConnection || !navigator.getUserMedia) {
  $scope.error = 'WebRTC is not supported by your browser. You can try the app with Chrome and Firefox.';
  return;
}
```

上記のコードは、WebRTCがサポートされているかを確認しています。
これは単純に`$scope.error`へ使えない旨のメッセージを設定して、コントローラの処理をさせないようにしています。

```javascript
var stream;
VideoStream.get()
.then(function (s) {
  stream = s;
  Room.init(stream);
  stream = URL.createObjectURL(stream);
  if (!$routeParams.roomId) {
    Room.createRoom()
    .then(function (roomId) {
      $location.path('/room/' + roomId);
    });
  } else {
    Room.joinRoom($routeParams.roomId);
  }
}, function () {
  $scope.error = 'No audio/video permissions. Please refresh your browser and allow the audio/video capturing.';
});
```

`VideoStream.get()`は最初にユーザーのメディアストリーム情報をresolvedするpromiseを返却します。
promiseがresolvedされると`Room`にStremを渡して初期化させます。
ユーザーのWebカメラで撮影した映像を見えるようにするため、`URL.createObjectURL`は、HTMLのvideo要素のsrcにその内容を設定する事が出来るようになる筈です。

次のステップとして、`roomid`が提供されているかどうかを確認してください。
提供されている場合は、roomid付きの部屋に参加しに行きます

```javascript
roomId`:`Room.joinRoom($routeParams.roomId);
```

そうでない場合、新しい部屋を作成します。
部屋が作られた後、クライアントはその部屋にリダイレクトされます。

`RoomCtrl`の残り２つのイベント処理についてです

```javascript
Room.on('peer.stream', function (peer) {
  console.log('Client connected, adding new stream');
  $scope.peers.push({
    id: peer.id,
    stream: URL.createObjectURL(peer.stream)
  });
});
Room.on('peer.disconnected', function (peer) {
  console.log('Client disconnected, removing stream');
  $scope.peers = $scope.peers.filter(function (p) {
    return p.id !== peer.id;
  });
});
```

- `peer.stream` - ピアStreamが受信されます。新しいピアStreamを受信し、ページ上で見えるようになっている`$scope.peers`配列に追加します。ページのマークアップはvideo要素にそれぞれの`stream`をマップします。
- `peer.disconnected` - ピアが切断されると `peer.disconnected`イベントが発生します。このイベントを受け取ったとき、接続のコレクションから該当のピアを削除することができます。

#### Room service

最後のコンポーネント`Room` serviceを作って行きましょう。

```bash
yo angular:factory Room
```

ファイル `/public/app/scripts/services/room.js` を編集して次の内容を設定して行きます。

```javascript
angular.module('publicApp')
  .factory('Room', function ($rootScope, $q, Io, config) {

    var iceConfig = { 'iceServers': [{ 'url': 'stun:stun.l.google.com:19302' }]},
        peerConnections = {},
        currentId, roomId,
        stream;

    function getPeerConnection(id) {
      if (peerConnections[id]) {
        return peerConnections[id];
      }
      var pc = new RTCPeerConnection(iceConfig);
      peerConnections[id] = pc;
      pc.addStream(stream);
      pc.onicecandidate = function (evnt) {
        socket.emit('msg', { by: currentId, to: id, ice: evnt.candidate, type: 'ice' });
      };
      pc.onaddstream = function (evnt) {
        console.log('Received new stream');
        api.trigger('peer.stream', [{
          id: id,
          stream: evnt.stream
        }]);
        if (!$rootScope.$$digest) {
          $rootScope.$apply();
        }
      };
      return pc;
    }

    function makeOffer(id) {
      var pc = getPeerConnection(id);
      pc.createOffer(function (sdp) {
        pc.setLocalDescription(sdp);
        console.log('Creating an offer for', id);
        socket.emit('msg', { by: currentId, to: id, sdp: sdp, type: 'sdp-offer' });
      }, function (e) {
        console.log(e);
      },
      { mandatory: { OfferToReceiveVideo: true, OfferToReceiveAudio: true }});
    }

    function handleMessage(data) {
      var pc = getPeerConnection(data.by);
      switch (data.type) {
        case 'sdp-offer':
          pc.setRemoteDescription(new RTCSessionDescription(data.sdp), function () {
            console.log('Setting remote description by offer');
            pc.createAnswer(function (sdp) {
              pc.setLocalDescription(sdp);
              socket.emit('msg', { by: currentId, to: data.by, sdp: sdp, type: 'sdp-answer' });
            });
          });
          break;
        case 'sdp-answer':
          pc.setRemoteDescription(new RTCSessionDescription(data.sdp), function () {
            console.log('Setting remote description by answer');
          }, function (e) {
            console.error(e);
          });
          break;
        case 'ice':
          if (data.ice) {
            console.log('Adding ice candidates');
            pc.addIceCandidate(new RTCIceCandidate(data.ice));
          }
          break;
      }
    }

    var socket = Io.connect(config.SIGNALIG_SERVER_URL),
        connected = false;

    function addHandlers(socket) {
      socket.on('peer.connected', function (params) {
        makeOffer(params.id);
      });
      socket.on('peer.disconnected', function (data) {
        api.trigger('peer.disconnected', [data]);
        if (!$rootScope.$$digest) {
          $rootScope.$apply();
        }
      });
      socket.on('msg', function (data) {
        handleMessage(data);
      });
    }

    var api = {
      joinRoom: function (r) {
        if (!connected) {
          socket.emit('init', { room: r }, function (roomid, id) {
            currentId = id;
            roomId = roomid;
          });
          connected = true;
        }
      },
      createRoom: function () {
        var d = $q.defer();
        socket.emit('init', null, function (roomid, id) {
          d.resolve(roomid);
          roomId = roomid;
          currentId = id;
          connected = true;
        });
        return d.promise;
      },
      init: function (s) {
        stream = s;
      }
    };
    EventEmitter.call(api);
    Object.setPrototypeOf(api, EventEmitter.prototype);

    addHandlers(socket);
    return api;
  });
```

`Room`は以下の依存関係を持ちます。

- `$rootScope` - `socket.io`イベントが受信される`$digest`のループイベントを呼び出す為に使われています。`socket.io`イベントハンドラが`$scope.$apply`内にラップされていないため、我々はマニュアルで`$digest`を起動する必要があります。
- `$q` - promiseベースのインタフェースを提供するため
- `Io` - `socket.io`のラッパー
- `config` - app.jsで定義されている定数

`Room` は以下のpublic APIを提供します。

```javascript
var api = {
  joinRoom: function (r) {
    if (!connected) {
      socket.emit('init', { room: r }, function (roomid, id) {
        currentId = id;
        roomId = roomid;
      });
      connected = true;
    }
  },
  createRoom: function () {
    var d = $q.defer();
    socket.emit('init', null, function (roomid, id) {
      d.resolve(roomid);
      roomId = roomid;
      currentId = id;
      connected = true;
    });
    return d.promise;
  },
  init: function (s) {
    stream = s;
  }
};
```

上で書いたように`joinRoom`は既存のルームに参加する為に使われます、`createRoom`は`Room`サービスを初期化する為に使われ
新しいチャットルームの作成の為に使われます。

このサービスで使う`socket.io`イベントは次の通りです。

- `peer.connected` - 新しいピアが部屋に参加したときに発火します。このイベントが発火したら、我々はこのピアに対して新しいSDP要求を開始します。
- `peer.disconnected` - ピアが切断されたときに発火
- `msg` - 新しいSDP申請/回答、またはICE候補を受信したときに発火

ピアが部屋に接続したとき、どのように新しい申請をするのかを見てみましょう。

```javascript
function makeOffer(id) {
  var pc = getPeerConnection(id);
  pc.createOffer(function (sdp) {
    pc.setLocalDescription(sdp);
    console.log('Creating an offer for', id);
    socket.emit('msg', { by: currentId, to: id, sdp: sdp, type: 'sdp-offer' });
  }, function (e) {
    console.log(e);
  },
  { mandatory: { OfferToReceiveVideo: true, OfferToReceiveAudio: true }});
}
```

新しいピアが部屋にやってくると、 `makeOffer` はピアのIDで呼び出されます。
まず最初にそのピアIDで`getPeerConnection`が実行されます。
指定されたピアID接続が存在しない場合、 `getPeerConnection`は新しい`RTCPeerConnection`を作成して必要なイベントハンドラを追加してから返します。

ピア接続を取得したら、 `createOffer` メソッドを呼びます。
このメソッドは`RTCPeerConnection`にある内容でSTUNサーバーへの新しい要求を行いつつ、ICE候補の収集を行います。
ICE候補やサポートされているコーデックなど、サーバに送信する為のSDP申請を作り上げます。

上にあるサーバーの実装で見たように、イベントオブジェクトの`to`プロパティが示すピアに申請をリダイレクトします。

次に`msg`イベントハンドラを見てみましょう。

```javascript
socket.on('msg', function (data) {
  handleMessage(data);
});
```
ここでは `handleMessage`を直接呼びます。次のように実装をトレースする事が出来ます。


```javascript
function handleMessage(data) {
  var pc = getPeerConnection(data.by);
  switch (data.type) {
    case 'sdp-offer':
      pc.setRemoteDescription(new RTCSessionDescription(data.sdp), function () {
        console.log('Setting remote description by offer');
        pc.createAnswer(function (sdp) {
          pc.setLocalDescription(sdp);
          socket.emit('msg', { by: currentId, to: data.by, sdp: sdp, type: 'sdp-answer' });
        });
      });
      break;
    case 'sdp-answer':
      pc.setRemoteDescription(new RTCSessionDescription(data.sdp), function () {
        console.log('Setting remote description by answer');
      }, function (e) {
        console.error(e);
      });
      break;
    case 'ice':
      if (data.ice) {
        console.log('Adding ice candidates');
        pc.addIceCandidate(new RTCIceCandidate(data.ice));
      }
      break;
  }
}
```

最初の行で、`by`プロパティがさすピアIDとのピア接続を取得します。
そして接続を取得後、異なるメッセージタイプに切り替えます。

- `SDP-offer` - このメッセージが表示された場合、この部屋の内部ピアのに参加している他の人々が、新しいピア接続を開始したいと申請してきている事を意味します。
ICE候補、ビデオコーデック、その他に答えるために `createAnswer`を使用して新しい回答を作成して、`setRemoteDescription`を行います。
SDPの回答を準備し、サーバーを経由して適切なピアを元に送信します。

- `SDP-answer` - 与えられたピアによってSDPの回答を受信して、元のピアにSDNの回答の返しを送ったことを意味しています。
リモートの状況を設定し、あとはP2P間でメディア接続を開始するだろうこと（対称的NATの後方ではない事）を願っています。

- `ice` - ネゴシエート中に新しいICE候補があれば、`RTCPeerConnection`インスタンスが現在ネゴシエートしている誰とのピアに新しい` msg`メッセージをリダイレクトし、`onicecandidate`イベントをトリガします。
その後`addIceCandidate`メソッドを使用して適切なピア接続にICE候補を追加していきます。

では、このチュートリアルの最後のメソッド`getPeerConnection`を見て行きましょう。

```javascript
var peerConnections = {};

function getPeerConnection(id) {
  if (peerConnections[id]) {
    return peerConnections[id];
  }
  var pc = new RTCPeerConnection(iceConfig);
  peerConnections[id] = pc;
  pc.addStream(stream);
  pc.onicecandidate = function (evnt) {
    socket.emit('msg', { by: currentId, to: id, ice: evnt.candidate, type: 'ice' });
  };
  pc.onaddstream = function (evnt) {
    console.log('Received new stream');
    api.trigger('peer.stream', [{
      id: id,
      stream: evnt.stream
    }]);
    if (!$rootScope.$$digest) {
      $rootScope.$apply();
    }
  };
  return pc;
}
```

このメソッドはピアIDと`RTCPeerConnection` オブジェクトのマッピングを作成する`peerConnections`オブジェクトを持ちます。
最初に与えられたピアID接続に関連する物があるかを確認し、あれば返します。
ピア接続が無い場合、イベントハンドラを追加した`onicecandidate`と`onaddstream`をキャッシュして返します。

`onaddstream`がトリガされると、接続が正常に開始された事を意味します。
以降 `peer.stream` イベントをトリガしてページ上のvideo要素でその内容を見えるようにすることができます。

#### ビデオプレイヤー

これがこのアプリケーション最後のコンポーネントです。

```bash
yo angular:directive videoPlayer
```

ファイル`public/app/scripts/directives/videoplayer.js`に以下の内容を設定してください。

```javascript
angular.module('publicApp')
  .directive('videoPlayer', function ($sce) {
    return {
      template: '<div><video ng-src="{{trustSrc()}}" autoplay></video></div>',
      restrict: 'E',
      replace: true,
      scope: {
        vidSrc: '@'
      },
      link: function (scope) {
        console.log('Initializing video-player');
        scope.trustSrc = function () {
          if (!scope.vidSrc) {
            return undefined;
          }
          return $sce.trustAsResourceUrl(scope.vidSrc);
        };
      }
    };
  });
```

## 終わりに

ではこれから、今まで作ってきた機能の振り返りを行いましょう。

### Full-mesh limitations

チュートリアルのデモから体験できるよう、このアプリケーションは１０未満のユーザーなら効率的に動きます。(ただしネットワーク帯域幅やCPUにもよっては５名)

これはfull-meshトポロジの制限です。
N対1のセッションを持っているとき、確立すべき`RTCPeerConnection`が`N-1`必要になります。
この中ではビデオストリームが`N-1`回エンコードされ、ネットワークを介して`N-1`回送信される事を意味します。

この処理は非常に非効率的であり、複数の当事者間の通信が必要とされるプロダクトだと非実用的です。
この問題の解決策は、WebRTCゲートウェイを使う事です。
そしてこの問題を解決する為のいくつかのオープンソースプロジェクトもあります。

 - [Jitsi Videobridge](https://jitsi.org/Projects/JitsiVideobridge) - Jitsi'sチームはシグナリングや接続の確立の為のColibri (Jitsi'sチームによるXMPP拡張)のためのXMPPシグナリングを使っており、WebRTC対応のビデオブリッジの環境を持っています。
ブリッジは低い計算能力を持つ安価なハードウェアを使った場合でも効果を十分発揮して、高い品質と転送用のビデオ/オーディオミキシングを提供します。

- [licode](http://lynckia.com/licode/) - シグナリングの為にビデオとオーディオのミキシングとカスタムJSONベースのプロトコルを提供するオープンソースプロジェクトです。私は最後にこれを使ってみましたが、このミキシングはオーディオ接続に使用する事はほぼ不可能なレベルの音質で、高い品質ではありませんでした。

### シグナリングプロトコル

このチュートリアルではシグナリングの為にカスタムJSONプロトコルを使用していました。
より良い選択はXMPPシグナルかSIPなどの標準化プロトコルを使う事でしょう。
この選択はあなたが他の既存サービスとサービスを統合する場合に、よりよい柔軟性を発揮します。

### その他

今回私たちがカバーしていない多くのトピックがありますが、残念な事にそれはこのチュートリアルの範囲外です。

さらに興味があるのであれば、以下のリソースを参考にしてください。
または、追加情報をpingしてください。

## Resources

- [HTML5 Rocks WebRTC intro](http://www.html5rocks.com/en/tutorials/webrtc/basics/)
- [HTML5 Rocks, WebRTC infrastructure](http://www.html5rocks.com/en/tutorials/webrtc/infrastructure/)
- [HTML5 Rocks, WebRTC data channel](http://www.html5rocks.com/en/tutorials/webrtc/datachannels/)
- [WebRTC.org](http://www.webrtc.org/)
- [WebRTC hacks](https://webrtchacks.com/)
