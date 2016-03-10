<!--
MEANスタックで今すぐ作る最新ウェブサービス:Twitter風サービスを作ってみよう
MEANスタックで今すぐ作る最新ウェブサービス:1時間で作れる！Twitter風サービス
フルスタック・Webサービスが今すぐ作れる！ - MEANスタック(1)
1時間でTwitter風サービスを作る方法 - MEANスタックで作る最新ウェブサービス
1時間でTwitter風フルスタック・Webサービスを作る！- MEANスタック開発(2)
0123456789012345678901234567890123456789012345678901234567890123
-->

<iframe src="https://player.vimeo.com/video/132998024?autoplay=1&title=0&byline=0&portrait=0" width="500" height="408" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>

<div style="text-align:right">(English article is <a href="http://engineering.paiza.io/entry/2015/07/09/154028">here</a>.)</div>

[f:id:paiza:20140712194904j:plain]こんにちは、吉岡([twitter:@yoshiokatsuneo])です。

[前回](http://paiza.hatenablog.com/entry/2015/07/08/最新・最速！Webサービスが今すぐ作れる！_-_MEANスタッ)はMEANスタックの一つ、YeomanのAngularJS Full-Stack generatorのインストールと使い方について紹介しました。MEANスタックはMongoDB, Express, AngularJS, Node.jsを組み合わせたWeb開発環境で、JavaScriptだけで簡単に素早く直感的で使いやすいフルスタック・Webサービスを作ることができます。

今回は、実際にもう少し実用的なWebサービスを作ってみましょう。

今回作るWebサービスは、Twitterのように投稿ができて、一覧表示ができるWebサービスです。(この記事のトップに有る動画で紹介しているサービスです)

ジェネレータが生成したコードを元にJavaScript/HTMLを修正するだけで、かなり本格的なものができますのでぜひ試してみてください。

[f:id:paiza:20150706130351p:plain]


[デモサイト: http://paizatter.herokuapp.com](http://paizatter.herokuapp.com)

今回作成するWebサービスは、以下のような機能を実装しています。

* サインアップ・ログイン
* メッセージの投稿、一覧表示、削除
* メッセージの検索
* 無限スクロールでの一覧表示
* お気に入りの設定・削除・一覧表示

なお、今回利用するコードは以下からダウンロードすることもできますが、一度は実際に手でコードを書き換えていくと雰囲気がつかみやすいです。

[https://github.com/gi-no/paizatter](https://github.com/gi-no/paizatter)

目次
====================
* [MEANスタックのインストール](#install)
* [プロジェクトの作成](#new_project)
* [メッセージ一覧表示](#list_message)
* [一覧表示の順番の変更](#change_order)
* [ユーザ認証](#user_authentication)
* [CSSの変更](#edit_css)
* [デプロイ](#deploy)
* [SNS認証](#sns_authentication)
* [デバッグ](#debug)
* [時刻表示フィルタ作成](#time_filter)
* [お気に入りの追加](#starred)
* [ユーザごとのメッセージ一覧・お気に入り一覧](#user_messages)
* [検索機能](#search)
* [無限スクロール](#infinite_scroll)
* [再デプロイ](#re_deploy)


<div id="install"></div>
■MEANスタックのインストール
=======================
YeomanのAngularJS Full-stack generator(generator-angular-fullstack)をインストールしていない場合、[前回の手順](http://paiza.hatenablog.com/entry/2015/07/08/最新・最速！Webサービスが今すぐ作れる！_-_MEANスタッ#install)でインストールしておいてください。

インストールしたAngularJS Full-stack generatorのバージョンが3.0.0以降であることを確認します。

```shell
$ npm ls -g generator-angular-fullstack
/usr/local/lib
└── generator-angular-fullstack@3.0.0-rc4 
```

古い場合、アップデートしてください。

```shell
$ sudo npm update -g generator-angular-fullstack
```

<div id="new_project"></div>
■プロジェクトの作成
=======================

まずはプロジェクトを作成します。

適当なディレクトリで、"yo"(Yeoman)コマンドを実行します。プロジェクト名は"paizatter"にしてみました。

```shell
$ mkdir paizatter
$ cd paizatter
$ yo angular-fullstack paizatter
```

ほぼデフォルト設定ですが、SNS認証は有効にしてみます。

```
- Would you like to include additional oAuth strategies? 
 ◉ Google
 ◉ Facebook
❯◉ Twitter
```

しばらく待つと、プロジェクト関連ファイルが作成されます。

今回関係する主なファイルは以下のとおりです。

```
.
|-- bower.json                            Bowerパッケージ一覧(クライアント側ライブラリ)
|-- package.json                          npmパッケージ一覧(サーバ側ライブラリ)
|
|-- client                                クライアント側コード
|   |-- app
|   |   |-- app.js                        クライアント側メインJavaScriptコード
|   |   `-- main
|   |       |-- main.controller.js        クライアント側コントローラ実装
|   |       |-- main.controller.spec.js   クライアント側テストコード
|   |       |-- main.html                 HTMLファイル
|   |       |-- main.js                   クライアント側ルーティング設定
|   |       `-- main.scss                 CSSファイル
|   |-- components
|   |   |-- navbar
|   |   |   |-- navbar.controller.js      Navbarコントローラ
|   |   |   `-- navbar.html               NavbarのHTMLファイル
|   |   `-- socket
|   |       `-- socket.service.js         クライアント側WebSocket実装
|   `-- index.html
|
`-- server                                サーバ側コード
    `-- api
        `-- thing
            |-- index.js                  サーバ側APIルーティング設定
            |-- thing.controller.js       サーバ側コントローラ(API実装)
            |-- thing.model.js            サーバ側DBモデル
            |-- thing.socket.js           サーバ側WebSocket実装
            `-- thing.integration.js      サーバ側テストコード
```

client配下にクライアント側のコードが配置され、server配下にサーバ側のコードが配置されています。

"client/app"配下では、各ページごとにディレクトリ(例: client/app/main)が作成され、そこにコントローラ、HTMLファイル(ビュー)、URLルーティング設定、CSSファイル、テストファイルがまとめて配置されます。ディレクトリごとに関連するファイルがまとまり管理しやすくなっています。

"server/api"配下も、ディレクトリ単位でコントローラ(API実装コード)、WebSocket関連コード、URLルーティング設定、テストコード、DBモデルがまとまっています。

[f:id:paiza:20150709025944p:plain]

クライアント側コントローラはサーバ側コントローラとAPIで通信しながらHTMLテンプレートを更新したりイベント処理します。サーバ側コントローラはクライアント側コントローラとAPIで通信しながらはモデルを通じてMongoDBのデータを取得・設定します。

MVCモデルに当てはめると、クライアント側から見るとサーバ側がモデルのようになり、サーバ側から見るとクライアントがビューのようになります。

Angular Full-stack generatorのデフォルトのnpmパッケージは少々古いので、npm-check-updatesでアップデートしておきます。

```shell
% sudo npm install -g npm-check-updates
% npm-check-updates -u
% npm install
```

準備ができたら、サーバを起動します。

```shell
% grunt serve
```


<div id="list_message"></div>
■メッセージ一覧表示
==============
作成されたプロジェクトは、"things"オブジェクトを一覧表示するプロジェクトになっています。

thingsオブジェクトをメッセージ一覧を保存するオブジェクトとして利用してみます。

まずは、入力ボックスと一覧表示のみできるように、HTMLファイルのcontainerクラスのdiv要素を書き換えてみます。
client/app/main/main.html:

```html
<div class="container">
  <br/>
  <form>
    <div class="input-group">
      <input type="text" class="form-control" placeholder="Message" ng-model="newThing">
      <span class="input-group-btn">
        <button type="submit" class="btn btn-primary" ng-click="addThing()">Add New</button>
      </span>
    </div>
  </form>

  <div class="row">
    <div ng-repeat="thing in awesomeThings">
      {{thing.name}}
    </div>
  </div>
</div>
```

入力フォームと、一覧が表示されます。

HTMLでは見慣れない、"ng-repeat"と"{{式}}"が使われていますが、これらがAngularJSでHTML中にJavaScript変数を表示するための記法です。

一覧表示で使われている"ng-repeat"は、配列の内容を表示するAngularJSの記法です。

"ng-repeat=ITEM in ARRAY"と書くことで、配列(ARRAY)の中身を繰り返して表示することができます。

また、"{{変数名}}"のように記述することで、変数名や簡単な式(Angular式)をHTML中に表示することができるようになります。

[f:id:paiza:20150706134637p:plain]


<div id="change_order"></div>
■一覧表示の順番の変更
================

表示はできましたが、新しいメッセージが下に追加されていますので、Twitterのように上に追加してみます。また、全てのメッセージを表示していますので、これを最新20件のみ表示するようにしてみます。

WebSocketのコードで、新しいアイテムが追加されたときに先頭に追加するため、"push"の代わりに"unshift"で配列オブジェクトに追加します。

client/components/socket/socket.service.js:

```javascript
      syncUpdates: ...
        socket.on(...
          ...
          // array.push(item);
          array.unshift(item);
```

上記socket.on()内のコード(コールバック)は、WebSocket経由で新しいオブジェクトの追加が通知された時に実行するコードになります。

WebSocketを使うことで、他のユーザがメッセージを追加したときにリロードボタンを押すことなく自動的に一覧表示が更新されます。


初回読み込み時やページの再読み込み時にメッセージ一覧を表示する時の順序も変えますので、サーバ側コントローラの一覧を返す部分も書き換えます。

server/api/thing/thing.controller.js:

```javascript
// Gets a list of Things
exports.index = function(req, res) {
  Thing.find().sort({_id:-1}).limit(20).execAsync()
    .then(responseWithResult(res))
    .catch(handleError(res));
};
```

まず、MongoDBのミドルウェアmongooseのsort()関数で作成時刻の降順でソートします。
MongoDBでは全てのドキュメント(RDBのレコード/行)に"_id"フィールドが含まれていおり、且つ"_id"フィールドの内容は時刻順になっていますので、"_id"をキーにソートすることで時刻順になります。

limit()関数で、表示数の上限を設定します。クエリの設定ができたら、exec()関数でクエリを呼び出します。クエリの結果はexecのコールバック関数の引数として受け取ることができますので、結果をそのままクライアントへ返します。


<div id="user_authentication"></div>
■ユーザ認証
=================
一覧表示はできましたが、これでは投稿ユーザが誰かわかりません。投稿の削除機能も、投稿ユーザのみ削除できるようにする必要があります。
ユーザ認証機能をつけてみましょう。

ユーザのサインアップ・ログイン機能自体は最初からテンプレートですでに作成されていますので、必要な機能にユーザ認証を設定し、メッセージ一覧でユーザ名を表示するようにします。


#### ◆サーバ側モデルスキーマ設定

メッセージを保存するときにユーザIDを一緒に保存するようにします。MongoDB自身はスキーマレスですが、Angular Full-stack generatorではmongooseというドライバを利用します。mongooseを利用することで、保存時に不必要なフィールドを保存しないようにしたり、フックしたり、関連するドキュメントを展開したり、といった便利な機能が利用できます。

まず、Mongooseスキーマ定義でメッセージ(ThingSchma)にユーザIDを追加します。

"name"フィールドでメッセージが保持し、userフィールドでユーザのObjectIdを保持します。userフィールドでは、"ref: 'User'"のようにしてObjectIdをUserコレクションと関連づけておくことで、後でpopulate()関数等で展開できるようになります。

また作成時刻も追加しておきます。
"createdAt"はdefaultとして、Date.now関数を指定することで、作成時刻が自動的に設定されるようにします。

server/api/thing/thing.model.js:

```javascript
var ThingSchema = new Schema({
  name: String, /* message */
  user: {
    type: Schema.ObjectId,
    ref: 'User'
  },
  createdAt: {
  	type: Date,
  	default: Date.now
  },
});
```


クエリ('find', 'findOne')に対しては、userフィールドはUserのオブジェクトIDを返すようになっています。実際のユーザ名を返すように、Userオブエジェクトに対してpopulate()を呼び出します。populateを呼び出すと、UserオブジェクトのIDではなく、Userオブジェクト自体が展開されるようになります。

populate('user')でUserオブジェクトの全てのフィールドが展開されますが、必要なフィールド('name')のみ展開するように、populate('user','name')と記述します。

個別のクエリでpopulate()を呼ぶこともできますが、すべてのクエリに対して展開するように、"pre()"でフックですべての'find', 'findOne'クエリに対してpopulate()を呼び出します。

```javascript
ThingSchema.pre('find', function(next){
  this.populate('user', 'name');
  next();
});
ThingSchema.pre('findOne', function(next){
  this.populate('user', 'name');
  next();
});
```


#### ◆サーバAPIのルーティング設定
認証が必要なリクエストに対しては、ルーティング設定で認証用のauth.isAuthenticated()ミドルウェアを使うように設定します。これにより、認証されていないユーザからの投稿や削除等を禁止するとともに、リクエストオブジェクト(req)のuserフィールド(req.user)にユーザオブジェクトが設定されます。

server/api/thing/index.js:

```javascript
var auth = require('../../auth/auth.service');

router.get('/', controller.index);
router.get('/:id', controller.show);
router.post('/', auth.isAuthenticated(), controller.create);
router.delete('/:id', auth.isAuthenticated(), controller.destroy);
```
上記ルーティングファイルでは、指定URLアクセス時に呼び出されるコントローラ関数を指定します。

認証を追加するため、post/deleteアクションで"auth.isAuthenticated()"をミドルウェアとして指定してます。編集用のput/patchは今回は使わないのでルーティングごと削除しておきます。


#### ◆サーバ側コントローラのcreate関数変更
サーバ側コントローラでは、オブジェクト作成時に作成するドキュメントのuserフィールドにユーザを設定します。

req.userはリクエスト時のユーザ情報がすでに設定されているので、このreq.userを"Thing.create"時の引数(req.body)に追加するだけでユーザ情報が保存されるようになります。

server/api/thing/thing.controller.js:

```javascript
// Creates a new Thing in the DB
exports.create = function(req, res) {
  req.body.user = req.user;
  Thing.createAsync(req.body)
    ...
```


#### ◆サーバ側コントローラの削除関数変更
削除時は、削除前に投稿メッセージのユーザと一致するか比較・検証します。

server/api/thing/thing.controller.js:

```javascript
function handleUnauthorized(req, res) {
  return function(entity) {
    if (!entity) {return null;}
    if(entity.user._id.toString() !== req.user._id.toString()){
      res.send(403).end();
      return null;
    }
    return entity;
  }
}
...
// Deletes a Thing from the DB
exports.destroy = function(req, res) {
  Thing.findByIdAsync(req.params.id)
    .then(handleEntityNotFound(res))
    .then(handleUnauthorized(req, res))
    .then(removeEntity(res))
    .catch(handleError(res));
};
```


#### ◆クライアント側コントローラの変更
自分のツイートかどうかわかるように、isMyTweet関数を追加します。

client/app/main/main.controller.js:

```javascript
angular.module('paizatterApp')
  .controller('MainCtrl', function ($scope, $http, socket, Auth) {
    $scope.isLoggedIn = Auth.isLoggedIn;
    $scope.getCurrentUser = Auth.getCurrentUser;
...
    $scope.isMyTweet = function(thing){
      return Auth.isLoggedIn() && thing.user && thing.user._id===Auth.getCurrentUser()._id;
    };
  });
```
上記コントローラ関数では、引数に指定したモジュールが利用できますので"Auth"を追加しました。また、$scope変数(連想配列)に関連付けた変数・関数はHTMLファイル中で参照できますので、$scope.isMyTweetに関数を作成します。

isMyTweetでは、メッセージ(thing)のユーザIDと、ログインユーザのユーザIDが一致するか比較しています。

また、認証関数が使えるように、isLoggedIn/getCurrentUserも$scope変数に追加しておきます。


#### ◆クライアント側HTMLの変更
一覧表示部分で、ユーザ名と日時を表示するようにしてみます。

client/app/main/main.html:

```html
  <div ng-repeat="thing in awesomeThings">
    <div class="row">
      {{thing.user.name}} - {{thing.name}} ({{thing.createdAt}})
      <button ng-if="isMyTweet(thing)" type="button" class="close" ng-click="deleteThing(thing)">&times;</button>
    </div>
  </div>
```

#### ◆サーバ側テストの変更
ルーティングのテストは今回は削除しておきます。

```shell
% rm server/api/thing/index.spec.js
```

また、認証が必要なAPIについては、"server/api/user/user.integration.js"を参考に、テスト前時にログインして認証情報を設定します。
PUT APIは利用しないので削除しておきます。

server/api/thing/thing.integration.js:

```javascipt
var User = require('../user/user.model');
...
describe('Thing API:', function() {
  var user;
  before(function() {
    return User.removeAsync().then(function() {
      user = new User({
        name: 'Fake User',
        email: 'test@test.com',
        password: 'password'
      });

      return user.saveAsync();
    });
  });

  var token;
  before(function(done) {
    request(app)
      .post('/auth/local')
      .send({
        email: 'test@test.com',
        password: 'password'
      })
      .expect(200)
      .expect('Content-Type', /json/)
      .end(function(err, res) {
        token = res.body.token;
        done();
      });
  });
  ...    
  describe('POST /api/things', function() {
    ...    
        .post('/api/things')
        .set('authorization', 'Bearer ' + token)
    ...
  describe('DELETE /api/things/:id', function() {
    ...
        .delete('/api/things/' + newThing._id)
        .set('authorization', 'Bearer ' + token)
    ...
        .delete('/api/things/' + newThing._id)
        .set('authorization', 'Bearer ' + token)
    ...
  /* describe('PUT /api/things/:id', function() {
  }); */
```


#### ◆テスト
認証部分は完成しました。ログインせずに投稿しようとすると、認証画面に移動しますのでサインアップしてください。

メッセージを投稿すると、投稿ユーザ名が表示されます。また自分の投稿のみバツボタンで削除できます。

[f:id:paiza:20150706135436p:plain]


<div id="edit_css"></div>
■CSSの変更
============
一覧表示のメッセージに飾りがないので、CSSでメッセージが表示されている雰囲気に変えてみます。


#### ◆CSSARROWの設定
http://cssarrowplease.com で吹き出しのCSSを選んでみます。

適当に選んだら、CSSファイルをコピーして、main.scssに追加します。

client/app/main/main.scss:

```css
// http://cssarrowplease.com
.arrow_box {
    position: relative;
    background: #f0f0f0;
    border: 4px solid #c2e1f5;
}
.arrow_box:after, .arrow_box:before {
    right: 100%;
    top: 50%;
    border: solid transparent;
    content: " ";
    height: 0;
    width: 0;
    position: absolute;
    pointer-events: none;
}

.arrow_box:after {
    border-color: rgba(224, 224, 224, 0);
    border-right-color: #f0f0f0;
    border-width: 10px;
    margin-top: -10px;
}
.arrow_box:before {
    border-color: rgba(194, 225, 245, 0);
    border-right-color: #c2e1f5;
    border-width: 16px;
    margin-top: -16px;
}
```

マージン等も設定してみます。

client/app/main/main.scss:

```css
.tweet{
    margin: 5px;
}
.arrow_box .message {
    font-size: 16px;
    height: 2em;
}
```


#### ◆HTMLファイルの変更
HTMLファイルをCSSを適用するように書き換えます。

client/app/main/main.html:

```html
  <div ng-repeat="thing in awesomeThings" class="tweet">
    <div class="row">
      <h2 class="col-xs-2">
        {{thing.user.name}}
      </h2>
      <div class="arrow_box col-xs-10">
        <button ng-if="isMyTweet(thing)" type="button" class="close" ng-click="deleteThing(thing)">&times;</button>
        <h2 class="message">
          {{thing.name}} 
        </h2>
        <span style="float: right;">({{thing.createdAt}})</span>
      </div>
    </div>
  </div>
```

[f:id:paiza:20150706140631p:plain]


<div id="deploy"></div>
■デプロイ
================
ここまでで基本的な機能は完成です。一旦デプロイしてみましょう。

```shell
% yo angular-fullstack:heroku
% cd dist
% heroku addons:add mongolab
```

"yo angular-fullstack:heroku"を実行することで、Herokuへのデプロイ環境が構築できます。

また、MongoDBモジュールも入れておきます。MongoHQとMongoLabがMongoDBアドオンとして利用できますが、無料プランがあるMongoLabを入れます。

以上で設定ができました。

次回以降のデプロイは"grunt"コマンドでのビルドと、"grunt buildcontrol:heroku"コマンドでのデプロイで行えます。

```shell
% grunt
% grunt buildcontrol:heroku
```

実際にブラウザを開いて表示してみましょう！

http://アプリケーション名.herokuapp.com/


<div id="sns_authentication"></div>
■SNS認証
========
SNS認証(Facebook, Twitter, Google)を利用する場合、APIキーとSECRETキーの登録を行います。登録手順は[前回](http://paiza.hatenablog.com/entry/2015/07/08/最新・最速！Webサービスが今すぐ作れる！_-_MEANスタッ#sns_link)を参照ください。


<div id="debug"></div>
■デバッグ
=========
デプロイしても動作しない場合は、ログを確認してみましょう。デバッグ方法については、[前回](http://paiza.hatenablog.com/entry/2015/07/08/最新・最速！Webサービスが今すぐ作れる！_-_MEANスタッ#debug)も参照ください。

```shell
% cd dist
% heroku logs
```

また、MongoDBの操作は、MongoHubなどのGUIツールを使うと便利です。MongoDBのURLはHerokuから取得します。

% heroku config
...
MONGOLAB_URI:    mongodb://ユーザ名:パスワード@ホスト名:ポート番号/データベース名
...


<div id="time_filter"></div>
■時刻表示フィルタ作成
==============

メッセージの作成時刻はUTCで表示されていますが、Twitterのように今からどれぐらい前に作成されたか表示するようにしてみましょう。


#### ◆momentjsのインストール

momentjsというライブラリを使います。クライアント側のライブラリ管理コマンドのbowerでインストールします。

```shell
% bower install --save momentjs
```

bowerの--saveオプションを指定することで、"bower.json"にパッケージ名が保存され、gruntにより自動的にindex.htmlにmomentjsを読み込むためのスクリプトタグが追加されます。


#### ◆fromNowフィルタ作成

"fromNow"という名前をAngularJSフィルタを作成します。フィルタは表示形式を変えるためのAnguarJSの機能で、ここでは現在からの時間で表示するフィルタを作成します。

ジェネレータでfromNowフィルタを作成します。ジェネレータはfromNowディレクトリを作成し、JavaScriptコードとテストコードをそのディレクトリに作ります。

また、現状、新規ディレクトリ作成後は、"grunt injector"を実行するか、"grunt serve"を実行してディレクトリ内のJavaScriptファイルが読み込まれるようにする必要があります。( [grunt-contrib-watch/issues/166](https://github.com/gruntjs/grunt-contrib-watch/issues/166) )

```shell
% yo angular-fullstack:filter fromNow
% grunt injector
```

フィルタの中身を、momentjsのfromNow関数を呼ぶように変えます。

client/app/fromNow/fromNow.filter.js

```javascript
    return function (input) {
      return moment(input).fromNow();
    };
```

fromNowフィルタは完成です。

それでは、fromNowフィルタを使ってみます。フィルタはHTMLでの変数埋め込み部分({{変数}})の最後に"|フィルタ名"と書くことで使えます。ここでは"{{thing.createdAt}}"を、"{{thing.createdAt|fromNow}}"と書き換えます。

client/app/main/main.html:

```html
        <span style="float: right;">({{thing.createdAt|fromNow}})</span>
```

これで、メッセージ作成時間が、現在からの時間で"〜minutes ago"のように表示されるようになりました。

[f:id:paiza:20150706145049p:plain]


#### ◆テストコード変更

また、フィルタと同時にテストコードも作成されていますが、コードを書き換えてためにテストが失敗してしまいますので、修正しておきます。


現在時刻に対して、'a few seconds ago'が帰ってくることを確認します。

client/app/fromNow/fromNow.filter.spec.js:


```javascript
  it('return "a few seconds ago" for now', function () {
    expect(fromNow(Date.now())).toBe('a few seconds ago');
  });
```

実際にテストしてみましょう。エラーがなければOKです。

```shell
% grunt test
```


#### ◆日本語(他言語)対応
日本語も表示する場合は、"moment-with-locales.min.js"を使います。client/index.htmlで"<!-- endbower -->"の下にscriptタグを追加します。

client/index.html

```html
      <!-- endbower -->
      <script src="bower_components/momentjs/min/moment-with-locales.min.js"></script>
```

fromNowフィルタで、ブラウザの言語(window.navigator.language)を使うようにします。

client/app/fromNow/fromNow.filter.js

```javascript
    return function (input) {
      return moment(input).locale(window.navigator.language).fromNow();
    };
```

これで、「〜分前」のように表示されるようになりました。

[f:id:paiza:20150706145342p:plain]


<div id="starred"></div>
■お気に入りの追加
===============

メッセージをお気に入りできるようにしてみましょう。


#### ◆サーバDBモデルで、メッセージのスキーマにお気に入りユーザを追加

各メッセージに、お気に入りしたユーザ一覧を保持するようにします。

MongoDBでは、配列をそのまま保持することができますので、各メッセージごとに、お気に入りしたユーザ一覧を保持します。メッセージのスキーマ定義でユーザのObjectID一覧を配列 として保持するstarsフィールドを追加します。

server/api/thing/thing.model.js:


```javascript
var ThingSchema = new Schema({
...
  stars: [{
    type: Schema.ObjectId,
    ref: 'User'
  }],
});
```


#### ◆サーバ側ルーティングの追加

お気に入りの追加・削除を行えるように、APIを２つ(star/unstar)サーバ側URLルーティングに追加します。認証ユーザのみ追加・削除できるように、isAuthenticatedをExpressのルーティングミドルウェアに追加します。

server/api/thing/index.js:

```
router.put('/:id/star', auth.isAuthenticated(), controller.star);
router.delete('/:id/star', auth.isAuthenticated(), controller.unstar);
```


#### ◆サーバ側APIの実装

star/unstar APIをサーバ側コントローラで実装します。

update関数に"{$push/$pull: {フィールド名: 値}}"とすることで、配列に値を追加したり削除したりできます。API実装は、"$push", "$pull"が違うだけで中身は全く同じです。データ更新後はshow()でドキュメントを返すようにしておきます。


server/api/thing/thing.controller.js:

```javascript
exports.star = function(req, res) {
  Thing.update({_id: req.params.id}, {$push: {stars: req.user._id}}, function(err, num){
    if (err) { return handleError(res)(err); }
    if(num===0) { return res.send(404).end(); }
    exports.show(req, res);
  });
};
exports.unstar = function(req, res) {
  Thing.update({_id: req.params.id}, {$pull: {stars: req.user._id}}, function(err, num){
    if (err) { return handleError(res)(err); }
    if(num === 0) { return res.send(404).end(); }
    exports.show(req, res);
  });
};
```


#### ◆クライアント側コントローラの実装

クライアント側コントローラでstar()/unstar()関数を作ります。これらの関数は、サーバAPIのstar/unstarを呼び出します。

こちらも呼び出しメソッドが"put"と"delete"で違う以外全く同じです。また、お気に入りしたメッセージかどうか確認するための関数isMyStarも作成しておきます。これは、自分のユーザIDがメッセージのお気に入り(stars)に含まれているか確認することで行います。

client/app/main/main.controller.js:


```javascript
    $scope.starThing = function(thing) {
      $http.put('/api/things/' + thing._id + '/star').success(function(newthing){
        $scope.awesomeThings[$scope.awesomeThings.indexOf(thing)] = newthing;
      });
    };
    $scope.unstarThing = function(thing) {
      $http.delete('/api/things/' + thing._id + '/star').success(function(newthing){
        $scope.awesomeThings[$scope.awesomeThings.indexOf(thing)] = newthing;
      });
    };
    $scope.isMyStar = function(thing){
      return Auth.isLoggedIn() && thing.stars && thing.stars.indexOf(Auth.getCurrentUser()._id)!==-1;
    };
```


#### ◆HTMLファイルの変更

HTMLファイルを変更して、星アイコンを追加し、クリックされたらstarThing()を呼び出すようにします。お気に入りされている場合は、中身が塗りつぶされた星マークにしてクリックされたらunstarThing()を呼び出します。

client/app/main/main.html:

```html
      ...
      <div class="arrow_box col-xs-10">
        <button ng-if="isMyTweet(thing)" type="button" class="close" ng-click="deleteThing(thing)">&times;</button>
        <button ng-if=" isMyStar(thing)" type="button" class="close" ng-click="unstarThing(thing)">
          <span class="glyphicon glyphicon-star" style="color: #CF7C00;" ></span>
        </button>
        <button ng-if="!isMyStar(thing)" type="button" class="close" ng-click="starThing(thing)"  >
          <span class="glyphicon glyphicon-star-empty"></span>
        </button>
        ...
```

以上でお気に入りができるようになりました。


<div id="user_messages"></div>
■ユーザごとのメッセージ一覧・お気に入り一覧
=================================

[f:id:paiza:20150706145600p:plain]

今までのメッセージ一覧では、すべてのユーザのメッセージを表示していましたが、自分や他のユーザのメッセージのみ、又は自分のお気に入りのみも表示できるようにしてみましょう。


#### ◆クライアントのルーティング追加
まずは、ユーザごとのメッセージ一覧やお気に入り一覧に対応するURLを以下のように作成します。

* ユーザのメッセージ一覧: /users/ユーザID
* ユーザのお気に入り一覧: /users/ユーザID/starred

ルーティング設定は、$stateProvider.state関数で行いますので、上記URLを追加します。表示内容は同じですので、同じコントローラ(MainCtrl)、同じテンプレート(main.html)を使うようにします。ただし、表示内容を絞り込むので、絞り込むためのクエリを指定します。"resolve:"フィールドの中に"query"のように指定することで、コントローラの引数で、query変数が使えるようになります。ユーザ一覧ではuserで、お気に入り一覧ではstarsで絞り込みます。

MongoDBではJavaScriptオブジェクトでQueryをかけるので、このqueryをサーバAPIを経由してMongoDBまで流し込めば検索ができます。

なお、"/users/:userId"をルーティングで先に書いてしまうと、"starred"もuserIdの一部となってしまうので、"/users/:userId/stared"を先に指定します。

client/app/main/main.js:

```javascript
angular.module('paizatterApp')
  .config(function ($stateProvider) {
    $stateProvider
      .state('main', {
        url: '/',
        templateUrl: 'app/main/main.html',
        controller: 'MainCtrl',
        resolve: {
          query: function(){return null;}
        },
      })
      .state('starred', {
        url: '/users/:userId/starred',
        templateUrl: 'app/main/main.html',
        controller: 'MainCtrl',
        resolve: {
          query: function($stateParams){
            return {stars: $stateParams.userId};
          }
        }
      })
      .state('user', {
        url: '/users/:userId',
        templateUrl: 'app/main/main.html',
        controller: 'MainCtrl',
        resolve: {
          query: function($stateParams){
            return {user: $stateParams.userId};
          }
        }
      })
      ;
  });
```


#### ◆クライアント側コントローラの変更

ルーティングで追加したquery変数をサーバに渡します。controller関数の引数にqueryを追加し、$http.getの引数として渡すだけです。

client/app/main/main.controller.js:

```javascript
  .controller('MainCtrl', function ($scope, $http, socket, Auth, query) {
  ...
    $http.get('/api/things', {params: {query: query}}).success(function(awesomeThings) {
```


#### ◆サーバ側コントローラの変更

サーバ側コントローラでは、受け取ったQueryをfind()の引数としてMongoDBに渡すだけです。

server/api/thing/thing.controller.js

```javascript
exports.index = ...
  var query = req.query.query && JSON.parse(req.query.query);
  Thing.find(query).sort...
```


#### ◆Navbarへのnavリンク追加

Navbarのリンクは"Home"だけなので、これを"All", "Mine", "Starred"(お気に入り)の３つにしてみます。

Navbarコントローラでリンクを$scope.menu配列に追加します。また、"Mine", "Starred"リンク先はログイン後のみ有効にします。ログイン前・ログイン後でリンクを動的に切り替えるため、リンク先URL用のフィールド('link')とリンク表示用のフィールド('show')項目は関数にします。

client/components/navbar/navbar.controller.js:

```javascript
    $scope.menu = [
      {
        'title': 'All',
        'link': function(){return '/';},
        'show': function(){return true;},
      },
      {
        'title': 'Mine',
        'link': function(){return '/users/' + Auth.getCurrentUser()._id;},
        'show': Auth.isLoggedIn,
      },
      {
        'title': 'Starred',
        'link': function(){return '/users/' + Auth.getCurrentUser()._id + '/starred';},
        'show': Auth.isLoggedIn,
      },
    ];
```

NavbarのHTMLファイルで、"link"を"link()"と関数呼び出しにします。"ng-show"ではitem.show()と指定し、show()がtrueの時のみ表示されるようにします。

client/components/navbar/navbar.html:

```html
        <li ng-repeat="item in menu" ng-class="{active: isActive(item.link())}" ng-show="item.show()">
            <a ng-href="{{item.link()}}">{{item.title}}</a>
        </li>

```


#### ◆クライアント側HTMLの変更
ユーザをクリックしたらユーザのツイート一覧を見えるようにしてみましょう。各ユーザ用のメッセージ一覧URL(/users/ユーザID)へのリンクを貼るだけです。

client/app/main/main.html:

```html
        <a ng-href="/users/{{thing.user._id}}">{{thing.user.name}}</a>
```


#### ◆テストコードの変更
最後に、テストコードでダミーのqueryパラメータを追加しておきます。

client/app/main/main.controller.spec.js:

```javascript
    MainCtrl = $controller('MainCtrl', {
      $scope: scope,
      query: null,
    });
```

これで、自分や他のユーザのメッセージ一覧や、お気に入り一覧を表示できるようになりました。


<div id="search"></div>
■検索機能
============

[f:id:paiza:20150706145744p:plain]

メッセージの検索機能も付けてみましょう。MongoDBには全文検索機能がありますので、これを利用してみます。


#### ◆クライアント側ルーティングの変更　

まず、検索キーワード"keyword"をキーとしたURLを割り当て、このURLに遷移することで検索できるようにします。

* 全体: /?keyword=検索キーワード
* ユーザ単位: /users/:userId?keyword=検索キーワード
* お気に入り: /users/:userId/starred?keyword=キーワード

ルーティング設定で"url: XXX?keyword"と記載して、キーワードをパラメータとして扱えるようにします。

client/app/main/main.js

```javascript
      .state('main', {
        url: '/?keyword',
      ...
      .state('starred', {
        url: '/users/:userId/starred?keyword',
      ...
      .state('user', {
        url: '/users/:userId?keyword',
```
      

#### ◆Navbarへの検索ボックス追加

Navbarに検索ボックスを追加します。検索実行時には"search(keyword)"関数を呼び出すように、ng-submitで設定します。

```html
    <div collapse="isCollapsed" class="navbar-collapse collapse" id="navbar-main">
      ...
      <form class="navbar-form navbar-left" role="search" ng-submit="search(keyword)">
        <div class="input-group">
          <input type="text" class="form-control" placeholder="Search" ng-model="keyword">
          <span class="input-group-btn">
            <button type="submit" class="btn btn-default"><span class="glyphicon glyphicon-search" ></span></button>
          </span>
        </div>
      </form>

      <ul class="nav navbar-nav navbar-right">
      ...
```


#### ◆Navbarコントローラ:検索時の遷移

Navbarコントローラで、検索時に指定されたkeywordに遷移するようにします。

client/components/navbar/navbar.controller.js:


```javascript
    $scope.search = function(keyword) {
      $state.go('main', {keyword: keyword});        
    };
```

ただ、これだと必ずすべてのユーザのメッセージ検索になりますので、自分のメッセージ、またはお気に入りのメッセージからも検索できるようにします。すでに一覧ページにいる場合、同じ状態(ユーザ、お気に入り)で遷移し、それ以外はすべてのメッセージを表示する状態('main')に遷移します。'$state'変数が使えるように、NavbarCtrl関数の引数に'$state'の追加を忘れないようにしましょう。

client/components/navbar/navbar.controller.js:


```javascript
  .controller('NavbarCtrl', function ($scope, $location, Auth, $state) {
    $scope.search = function(keyword) {
      if ($state.current.controller === 'MainCtrl'){
        $state.go($state.current.name, {keyword: keyword}, {reload: true});        
      }else{
        $state.go('main', {keyword: keyword}, {reload: true});        
      }
    };
```


#### ◆正規表現での検索

まずは正規表現で検索してみましょう。MongoDBでは'$regex'を使って正規表現を指定します。

client/app/main/main.controller.js:

```javascript
  .controller('MainCtrl', function($scope, $http, $location, socket, Auth, query) {
    ...
    var keyword = $location.search().keyword;
    if(keyword){
      query = _.merge(query, {name: {$regex: keyword, $options: 'i'}});
    }
	$http.get('/api/things', {params: {query: query}})...
```


#### ◆全文検索

これで検索はできますが、検索のたびにすべてのメッセージを見ることになってしまい、メッセージが増えてくると遅くなってしまいます。
MongoDBの全文検索機能を使ってみましょう。全文検索では、'$text', '$search'キーワードを使って検索します。なお、フィールド名を指定しません。

client/app/main/main.controller.js:

```javascript
  .controller('MainCtrl', function($scope, $http, $location, socket, Auth, query) {
    ...
    var keyword = $location.search().keyword;
    if(keyword){
      query = _.merge(query||{}, {$text: {$search: keyword}});
    }
	$http.get('/api/things', {params: {query: query}})...
```


#### ◆サーバ側モデルの変更

全文検索を利用する場合、スキーマに'text'インデックスを設定します。

server/api/thing/thing.mode.js:

```javascript
ThingSchema.index({name: 'text'});
```

以上で検索できるようになりました。"Development"のようなキーワードで検索できます。(部分一致はできません。)


#### ◆日本語検索対応
MongoDBの全文検索エンジンは日本語の分かち書きに対応していません。

本格的な検索にはElasticSearchなども使えますが、ここではTinySegmenterを使ってみます。


"tokenizedName"フィールドを追加して分かち書き後のメッセージを保存することで、全文検索用のインデックスを作成するようにします。
```javascript
var ThingSchema = new Schema({
  name: String,
  tokenizedName: String,
  ...
}
...
ThingSchema.index({tokenizedName: 'text', name: 'text'});
```

テキストインデックスを作り直す場合、一度古いコレクションを削除しておきます。
```shell
% mongo
> use アプリケーション名-dev
> db.things.drop()
```


TinySegmenterをサーバ用のライブラリとしてnpmコマンドでインストールします。
```shell
% npm install --save r7kamura/tiny-segmenter
```

ドキュメントの保存時に分かち書きを行い、スペース区切りで連結してtokenizedNameに保存します。スキーマに対して、pre('save', 関数)と書くことで、保存の直前に指定の動作を行うことができます。

server/api/thing/thing.model.js:

```javascript
var TinySegmenter = require('tiny-segmenter');
...
ThingSchema.pre('save', function(next){
  var tinySegmenter = new TinySegmenter();
  this.tokenizedName = tinySegmenter.segment(this.name).join(' ');
  next();
});
```


これで日本語の分かち書きに対応できました。「吾輩は猫である」というメッセージに対して「我輩」で検索できます。


<div id="infinite_scroll"></div>
■無限スクロール
============

メッセージは最新の20件のみ表示されるようになっているため、それ以上古いメッセージを見ることができていません。

Twitterのように古いメッセージをスクロールで見れるようにしてみましょう。


#### ◆ライブラリ(ngInfiniteScroll)のインストール
無限スクロール用のAngularJSライブラリ(ngInfiniteScroll)をインストールします。

```javascript
% bower install --save ngInfiniteScroll
```


#### ◆ngInfiniteScrollの読み込み
ngInfiniteScrollモジュールを使えるように、AngularJSのアプリケーションモジュールの依存モジュールに追加します。

client/app/app.js:

```javascript
angular.module('paizatterApp', [
   ... ,
   'infinite-scroll'
]);
```


#### ◆HTMLファイルの変更

ngInfiniteScrollモジュールを利用するように、無限スクロールする部分(containerクラスのdiv要素)のタグにinfinite-scroll属性をに追加し、スクロール時に呼び出す関数(nextPage())を指定します。

読み込み中及び、もう読み込むメッセージがない場合はそれ以上スクロールしないようにinfinite-scroll-disabled属性でフラグ(busy, noMoreData)を指定します。

HTMLの最後では、読み込み中であることを表示します。

client/app/main/main.html:

```html
<div class="container" infinite-scroll='nextPage()' infinite-scroll-disabled='busy || noMoreData'>
  ...
  <div ng-show='busy'>Loading data...</div>
</div>
```


#### ◆クライアント側コントローラの変更

読み込み中かどうかを保存するbusy変数と、最後までデータを呼んだかどうかを保存するnoMoreData変数を作成します。

スクロール時には、すでに読み込んでいる最後のメッセージより古いメッセージのみ検索するように、クエリに条件"{_id: {$lt: lastId}}"を追加しています。

最初のメッセージ読み込み時は、指定件数(20)より少ない場合は読み込み済みにしておきます。

client/app/main/main.controller.js:


```javascript
    $scope.busy = true;
    $scope.noMoreData = false;
    ...
    $http.get('/api/things', ...
      ...
      if($scope.awesomeThings.length<20){
        $scope.noMoreData = true;
      }
      $scope.busy = false;
    });

    $scope.nextPage = function(){
      if($scope.busy){
        return;
      }
      $scope.busy = true;
      var lastId = $scope.awesomeThings[$scope.awesomeThings.length-1]._id;
      var pageQuery = _.merge(query, {_id: {$lt: lastId}});
      $http.get('/api/things', {params: {query: pageQuery}}).success(function(awesomeThings_) {
        $scope.awesomeThings = $scope.awesomeThings.concat(awesomeThings_);
        $scope.busy = false;
        if(awesomeThings_.length === 0){
          $scope.noMoreData = true;
        }
      });
    };

```

以上で20件以上のデータは無限スクロールで読み込めるようになりました。


テスト結果が問題ないことを確認します。

```shell
% grunt test
```


<div id="re_deploy"></div>
■再デプロイ
===============

以上で一通り開発できましたので、Herokuに再デプロイしてみましょう。

```shell
% grunt
% grunt buildcontrol:heroku
```

ブラウザで開いて確認しましょう！

http://アプリケーション名.herokuapp.com/


<div id="summary"></div>
■まとめ
=================
今回はMEANスタック開発環境Angular Full-stack generatorを使って、フルスタックのTwitter風Webサービスを作ってみました。

少ないコードですが、ユーザ認証を含む本格的なWebサービスが作れました。

今回の説明をベースにすることで、簡単なWebサービスでしたらかなり実用的なものまで作れるかと思います。

MEANスタックでは、このようにJavaScriptだけで気軽にWebサービスを作ることができます。ぜひアイデアを生かして、思いついたWebサービスを気軽に作ってみてはいかがでしょうか？

手順通りでも動作しないなど、気づいた点がありましたら、コメント等でフィードバックをいただけますとうれしいです。

今後も、MEANスタックを使った他のWebサービスの作り方の例も紹介していきたいと思います。

<table width="100%" border="0" cellspacing="0" cellpadding="0">
<thead>
  <tr>
    <td colspan="2" style="background-color: darkblue; color: white;"><div style="font-size: small; font-weight: bold;">MEANスタック開発記事一覧</div></td>
  </tr>
</thead>
<tbody>
  <tr>
    <td></td>
    <td><a href="http://paiza.hatenablog.com/entry/2015/07/08/最新・最速！Webサービスが今すぐ作れる！_-_MEANスタッ">最新・最速！Webサービスが今すぐ作れる！ - MEANスタック開発(1)</a></td>
  </tr>
  <tr>
    <td>→</td>
    <td><xa href="http://paiza.hatenablog.com/entry/2015/07/09/1時間でTwitter風フルスタック・Webサービスを作る！-_MEANス">初級者でも1時間でTwitter風Webサービスを作れる！- MEANスタック開発(2)</xa></td>
  </tr>
  <tr>
    <td></td>
    <td><a href="http://paiza.hatenablog.com/entry/meanstack_howto_3">Webサービスを作りたい人に最適、たった1時間でJSベースのQAサイトを作る方法 - MEANスタック開発(3)</a></td>
  </tr>

</tbody>
</table>


<hr>


<a href="http://paiza.jp/">paiza</a>ではITエンジニアとしてのスキルレベル測定(9言語に対応)や、プログラミング問題による学習コンテンツ(<a href="https://paiza.jp/learning"  target="_blank">paiza Learning</a>)を提供(こちらは21言語に対応)しています。テストの結果によりS,A,B,C,D,Eの６段階でランクが分かります。自分のプログラミングスキルを客観的に知りたいという方は是非チャレンジしてみてください。
<!--img src="http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20140901/20140901151109.jpg"></a-->
<a href="http://paiza.jp"><img src="http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20130917/20130917190908.gif?1379412762"></a>

<hr>
