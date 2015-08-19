<!--
MEANスタックで今すぐ作る最新ウェブサービス:ジェネレータを使ってみよう編
フルスタック・ウェブ開発環境の使い方 - MEANスタックで作る最新ウェブサービス
フルスタック・Webサービスが今すぐ作れる！ - MEANスタック開発(1)
最新・最速！Webサービスが今すぐ作れる！ - MEANスタック開発(1)
0123456789012345678901234567890123456789012345678901234567890123
-->
<!--div class="paiza-custom-header"-->
[f:id:paiza:20150703174202p:plain]

<div style="text-align:right">(English article is <a href="http://engineering.paiza.io/entry/2015/07/08/153011">here</a>.)</div>

[f:id:paiza:20140712194904j:plain]こんにちは、吉岡([twitter:@yoshiokatsuneo])です。


最近のリッチなWebサービス開発ではブラウザ(クライアント)とサーバ両方のコードを書いたり、Webソケットなどで連携したりすることもあり、気軽に取り組みづらくなっています。しかし、MEANスタックはクライアントからサーバまで全てをまとめた環境で、これを使えば素早くWebサービスを作ることができます。

今回は、MEANスタックの開発環境の一つ、AngularJS Full-stack generator(generator-angular-fullstack)を用いて実際にサンプルプログラムを動かしたり、修正したり、Herokuにデプロイしたりしてみます。

ブラウザ側のJavaScriptコードだけの作り方や、サーバ側だけのコードの書き方はWeb上でも多くでていますが、クライアントからサーバまで通して完全に動作するフルスタック・Webサービスを簡単に作れるのがMEANスタックの特徴です。この便利さをぜひ試してみてください。

次回以降の記事では、今回をベースに、さらに実用的なWebサービスの作り方にも取り組みたいと思います。



[AngularJS Full-stack generatorのデモサイト:http://fullstack-demo.herokuapp.com](http://fullstack-demo.herokuapp.com)

目次
=================
* [MEANスタックとは？](#about)
* [MEANスタックの特徴](#features)
* [インストール](#install)
* [プロジェクト作成](#new_project)
* [実行](#run)
* [コードの編集](#edit)
* [デバッグ](#debug)
* [デプロイ](#deploy)
* [SNSログイン設定](#sns_link)
* [まとめ](#summary)
* [参照](#reference)

<div id="about"></div>
■MEANスタックとは？ 
=======================

MEANスタックとは、MongoDB, Express, AngularJS, Node.jsを組み合わせることで、クライアント・サーバ・データベースの開発環境をパッケージ化した、Webサービス作成のためのフルスタック・フレームワークです。



[https://www.mongodb.org:image=https://upload.wikimedia.org/wikipedia/commons/e/eb/MongoDB_Logo.png]



[http://expressjs.com/:image=https://upload.wikimedia.org/wikipedia/commons/6/64/Expressjs.png]



[https://angularjs.org:image=https://raw.githubusercontent.com/angular/angular.js/master/images/logo/AngularJS.exports/AngularJS-medium.png]



[https://nodejs.org/:image=https://nodejs.org/images/logos/nodejs.png]



Webサービスを作るためのフレームワークとしては、従来CakePHP、Ruby on Railsなどが使われてきました。これらのフレームワークが用意しているルールに従うことで、Webサービスの開発をスピードアップし、ソースコードの見通しをよくし、大規模な開発にも対応できるようになってきました。

CakePHPやRuby on Railsは、サーバで動作するプログラムはサーバ上で動作してHTMLを生成し、ブラウザ側で表示するプログラムです。ブラウザ上でリンクをクリックしたり、フォームを送信したりすると、ブラウザは今表示している内容を捨てて、サーバ側で生成されたHTMLを表示します。

しかしながら、このようなサーバ側フレームワークだけでは、何か操作する度に待ちが入り、また現在の状態(入力内容、選択状態、スクロール位置など)がクリアされてしまうため、スムーズな動きはできません。

そこで、Google Mapsで代表されるようなAjaxが登場して以降、クライアント側でJavaScriptを用いてスムーズな動きを利用する場面が増えてきました。

最初はjQuery等のライブラリを使うことで、特定のUIのみでJavaScriptを使っていましたが、より多くのJavaScriptを使う中で、JavaScript側でもフレームワークが必要になってきました。現在、広く使われているJavaScriptフレームワークの1つがAngularJSです。

さらに、もともとWebページは、ブラウザ上での操作(リンクのクリックなど)の後にサーバ側でページを返す動作でした。そのため、ユーザが何も操作していないのにサーバ側から通知を行うことはできませんでした。これでは、チャットのようなサービスで新しいメッセージを通知したり、ゲーム等で相手の手を表示したりすることができません。

そこで考え出されたのは、WebSocketという技術で、これにより従来は不可能だったブラウザとWebサーバの双方向通知が可能になりました。また、WebSocketではサーバとブラウザが常時接続された状態になり、大量の接続に対応する必要があります。これを実現するためには、従来の単にブラウザのリクエストに応答するWebサーバとは別のサーバプログラムが必要になってきます。このような大量の接続に対応したサーバプログラムを実現する方法として考え出されたのが、node.jsです。node.jsではJavaScriptが持つ非同期処理を利用して、シングルスレッド上で複数のクライアント(ブラウザ)を管理できるようになっています。

このように、クライアント側でもJavaScript・サーバ側でもJavaScriptを利用できるようになりましたが、データベースもJavaScriptが直接利用できるMongoDBがあります。MongoDBはスキーマレスなので、マイグレーションコードを書かなくても、どんどん開発をすすめていくことができます。

MEANスタックは、このように、クライアント側のAngularJS, サーバ側のNode.js/Express.JS、データベースのmongodb、を組み合わせたJavaScriptベースのフルスタックのフレームワークです。

MEANスタック自体は単なる組み合わせの名前で、実際に利用するためのツールとしてはMEAN.IOやMEAN.JSなどもありますが、ここではジェネレータやテンプレートが充実していて取り組みやすいAngularJS Full-Stack generator(angular-fullstack-generator)で作っていきます。


<div id="features"></div>
■MEANスタックの特徴 (generator-angular-fullstack)
=======================

#### ◆フルスタック

クライアント・サーバ・データベースの組み合わせの面倒をみてくれるので、全体の見通しがよくなります。
特に、クライアントとサーバの連携やその環境設定が最初から行われているところは大変便利です。


#### ◆JavaScript単一言語
クライアント・サーバ・データベースまで、単一のJavaScriptのコードを書けます。
複数の言語を頻繁に行き来するとしなくていいというは、ストレスが少なく非常に大きなメリットです。


#### ◆一般的なツールを利用
MEANスタックで使われているMongoDB、Node.js、Express、AngularJSは、MEANスタックだけのために作られたわけではなく、広く使われているフレームワークです。また、generator-angular-fullstackで使われているツールであるYeoman、Bower、npm、Grunt、Karma、mochaも、他の開発環境でも広く使われているツールです。

このような広く使われているツール類を利用することで、安定した動作、豊富な利用例の活用、他の環境でのノウハウの活用などもできます。Webの開発環境は変化が早いですが、広く使われているツール類を使うことで、今後の変化に対応したり最新の技術に追随したりしやすくなります。


#### ◆ジェネレータ
generator-angular-fullstackでは、メニューで項目を選択しながらプロジェクトテンプレートを作成できます。以下の項目が設定できます。

* クライアント
  * スクリプト: JavaScript, CoffeeScript, Babel
  * マークアップ: HTML, Jade
  * スタイルシート: CSS, Stylus, Sass, Less
  * AnguarJSルーティング: ngRoute, ui-router

* サーバ
  * データベース: なし, MongoDB
  * 認証: あり、なし
  * oAuth連携: Facebook, Twitter, Google
  * Socket.io連携: あり、なし

さらに、クライアント側・サーバ側両方で、コントローラ等のテンプレートを作成するジェネレータも用意されています。ジェネレータを利用することで、簡単に開発をはじめることができます。HerokuやOpenShiftへのデプロイするためのコマンドも用意されており、簡単にサービスをリリースできます。


<div id="install"></div>
■インストール
=========================
まずは、MEANスタック用のツールであるAngularJS Full-Stack generatorをインストールします。

MEANスタック自体はMac、Linux、Windowsなどで動作しますが、ここではMac OS Xを前提として勧めます。他の環境の場合、適時読み替えていただければと思います。



* Node.jsをインストールします。(入っていない場合)

https://nodejs.org から「Install」ボタンでパッケージをダウンロードして実行します。

* Yeoman, Bower, Grunt, Gulpをインストールします。(入っていない場合)

```sh
% sudo npm install -g yo bower grunt-cli gulp
```

* AngularJS Full-Stack generator(generator-angular-fullstack)をインストールします。

```sh
% sudo npm install -g generator-angular-fullstack
```

* Homebewをインストールします。(入ってない場合)

```sh
% ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

* MongoDBをインストールします。(入ってない場合)

```sh
% brew update
% brew install mongodb
% ln -fs /usr/local/opt/mongodb/homebrew.mxcl.mongodb.plist ~/Library/LaunchAgents/
launchctl load ~/Library/LaunchAgents/homebrew.mxcl.mongodb.plist
```

* MongoDBが動いているか試してみましょう。

MongoDBはスキーマレスですので、SQLのcreate文のようなものは必要ありません。いきなりデータを追加することができます。

MongoDBでは、一つ一つのデータ(SQLのレコード/行)にあたるものをドキュメントと言い、任意のJSONオブジェクトが保存できます。データの集まり(SQLのテーブル)はコレクションと言います。

試しに、コレクション("my_collection")へのデータの追加、検索、削除を行ってみましょう。

```sh
~$ /usr/local/bin/mongo
MongoDB shell version: 3.0.2
connecting to: test
> db.my_collection.save( { 'hello' : 'world' } )
WriteResult({ "nInserted" : 1 })
> db.my_collection.find()
{ "_id" : ObjectId("558b92df5eed5e30d6c9c84f"), "hello" : "world" }
> db.my_collection.remove({})
WriteResult({ "nRemoved" : 1 })

```

なお、MongoDBの操作は、Mac OS Xでは[MongoHub](http://mongohub.todayclose.com)などのGUIツールを利用すると便利です。

[f:id:paiza:20150706155509p:plain]

[MongoHub](http://mongohub.todayclose.com)

<div id="new_project"></div>
■プロジェクト作成
=================
準備ができたらプロジェクトを作ってみましょう。

* プロジェクト用のディレクトリを用意して移動します。

```sh
% mkdir sample
% cd sample
```

* AngularJS Full-Stack generatorを使ってプロジェクトを作成します。

```sh
% yo angular-fullstack sample
```

質問が出てきますが、全てデフォルト(リターンキー)で問題ありませんが、以下のようなoAuthの質問でSNS認証を有効にできます。ここでは全て有効にしてみます。

```
- Would you like to include additional oAuth strategies? 
 ◉ Google
 ◉ Facebook
❯◯ Twitter

```

全ての質問に答えると、プロジェクトテンプレートが作成され、必要なモジュールのインストールが行われます。

特に初めて実行する場合は、時間がかかるかもしれません。コーヒーでも飲んで休憩しましょう。

* パッケージのアップデート
AngularJS Full-stack generatorのデフォルトのパッケージは少々古いので、npm-check-updatesでアップデートしておきます。

```shell
% sudo npm install -g npm-check-updates
% npm-check-updates -u
% npm install
```

<div id="run"></div>
■実行
==========

早速実行してみましょう。と言っても、以下のコマンドを打つだけです。

```sh
% grunt serve
```
各種タスクが実行されてnode.jsサーバが立ち上がり、ブラウザで http://localhost:9000 が開きます。

この時点で、既に以下のような機能が実装されています。

* Twitter BootstrapベースのUI
* 項目の追加・削除・一覧表示
* 他のユーザにより項目が追加された場合の、一覧のリアルタイムアップデート
* 認証機能(サインアップ・サインイン・SNS連携(APIキーの設定があれば))

実際に入力フォームに何か入力して、"Add New"を押すと一覧に追加されます。さらに、複数のブラウザを開いて、片方のブラウザで追加すると、もう片方の一覧も(リロードすることなく)リアルタイムでアップデートされてます。


<div id="edit"></div>
■コードの編集
================
さらに、"grunt serve"を実行した状態で、ファイルを変更してみましょう。

HTMLファイルを書き換えてみます。

app/main/main.html:

```html
<h1>こんにちは!</h1>
```
ファイルを保存した瞬間に、(ブラウザ側でリロードしなくても)ブラウザ上の表示がリアルタイムで更新されます。


<div id="debug"></div>
■デバッグ
=================

#### ◆クライアントプログラムのデバッグ
クライアントプログラムのデバッグは、ブラウザの開発ツールが利用できます。

Chromeの場合、URLバー右のChromeメニューアイコンをクリックし、[その他のツール][デベロッパーツール]を選択すると、デベロッパーツールが表示されます。

[f:id:paiza:20150708140450p:plain]

Safariの場合は、Safariメニューの環境設定の「拡張」で「メニューバーに"開発"メニューを表示」をチェックして開発メニューを有効にします。デバッグするページを開いた状態で、「開発」メニューの「エラーコンソールを表示」を選ぶことで、Chromeと同様の開発ツールが表示されます。

開発ツールでは、ログを確認したり、JavaScriptコードのブレークポイントを設定したり、変数を表示したりできます。


#### ◆サーバプログラムのデバッグ

サーバプログラムのデバッグは、Node.jsデバッガ(Node Inspector)を使ってみましょう。

Node InspectorはChromeで動作しますので、Gruntfile.jsでopenの行を以下のように書き換えて、ブザウザに"Google Chrome.app"を指定します。

Gruntfile.js:

```javascript
            // opens browser on initial server start
            nodemon.on('config:update', function () {
              setTimeout(function () {
                require('open')('http://localhost:8080/debug?port=5858', '/Applications/Google Chrome.app');
              }, 500);
            });
```

サーバをデバッグモードで起動します。

```sh
% grunt serve:debug
```

しばらくすると、Chromeのデベロッパーツールが立ち上がり、Node.jsのコードをデバッグできるようになります。最初の行で停止した状態になっていますので、Resumeボタン("F8")で実行を再開します。

この画面上で、ブレークポイントを設定したり、ステップ実行したり、変数の内容を確認したりできます。

[f:id:paiza:20150706114630p:plain]


<div id="deploy"></div>
■デプロイ
===========
"grunt serve"コマンドで起動した場合、サービスがローカルで動作しているため他の人が使うことはできません。他の人も使えるようにするために、プログラムをサーバにアップロードして実行します(デプロイ)。

AngularJS Full-stack generatorではHerokuに対応しており、このデプロイ作業をコマンドから行うことができるようになっています。


#### ◆Herokuアカウント作成

Herokuでは、1Webサービスに限り制限付きで無料でWebサービスを動作させることができます。Herokuのサイトに接続して、「Sign up for free」ボタンでアカウントを作っておきます。

https://www.heroku.com

[f:id:paiza:20150706114938p:plain]


#### ◆Heroku Toolbeltのインストール

HerokuでのWebサービスの管理はHeroku Toolbeltというコマンドラインツールを利用しますので、以下からダウンロード・インストールしておきます。

https://toolbelt.heroku.com

インストールしたら、herokuコマンドでログインしておきます。

```sh
% heroku login
```
ユーザ名とパスワードを聞かれるので入力します。

#### ◆WebサービスのHerokuデプロイ環境設定

Webサービスをデプロイするための設定を、Angular Full-stack generatorで行います。

"yo angular-fullstack:heroku"コマンドでHerokuへのデプロイ環境を設定できます。アプリケーション名を聞かれるので、適当な名前を入れておきます。

"http://アプリケーション名.herokuapp.com"でアクセスできるようになります。

また、MongoDBモジュールも入れておきます。MongoHQとMongoLabが利用できますが、無料プランがあるMongoLabを入れます。

```sh
% yo angular-fullstack:heroku
% cd dist
% heroku addons:add mongolab
```

以上で設定ができました。

次回以降のデプロイは、"grunt"コマンドでのビルドと"grunt buildcontrol:heroku"コマンドでのデプロイで行えます。

```sh
% grunt
% grunt buildcontrol:heroku
```

デプロイできました！ブラウザからデプロイしたWebサービスが表示されるか確認してみましょう。

http://アプリケーション名.herokuapp.com/

なお、エラーなどが表示された場合は、ログを見てみましょう。
```sh
% heroku logs
```


<div id="sns_link"></div>
■SNSログイン設定
==========
サインアップ・ログイン機能は実装されていますが、SNSログイン機能を使う場合、各SNSのAPIキー・SECRETキー設定を行ってください。

#### ◆Twitter

  https://apps.twitter.com から、「Create New App」でアプリケーションを作成します。"Callback URL"にはサービスのURL(例: http://paizatter.herokuapp.com )を指定します。

作成できたら、アプリケーションページの"Keys and Access Tokens"にて、"Consumer Key (API Key)"と"Consumer Secret (API Secret)"が確認できます。

[f:id:paiza:20150706115258p:plain]

#### ◆Facebook

  https://developers.facebook.com/apps/ から、「Add a New App」でアプリケーションを作成します。"Website"を選んでから、アプリケーションの名前を入力し、"Choose a Category"からカテゴリとして"Utilities"などを選び「Create App ID」ボタンで作成します。

  "My Apps"メニューから作成したアプリケーションを選び、"Advanced"タブの"Valid OAuth redirect URIs"でアプリケーションのURLを指定します。(例: http://アプリケーション名.herokuapp.com )。複数指定できるので、"http://localhost:9000/"なども入れておくとローカルでテストできるようになります。

[f:id:paiza:20150706115559p:plain]

  アプリケーションの"Settings"メニューで"Contact Email"を入力し、"Status & Review"から"Do you want to make this app and all its live features available to the general public?"で"Yes"を指定すると、アプリケーションが有効になります。これで、Dashboardから"App ID"と"App Secret"が入手できるようになっています。


#### ◆Google
  https://console.developers.google.com/project の「プロジェクトを作成」ボタンでプロジェクトを作成します。作成したプロジェクトを選択し、左のメニューから「APIと認証(APIs&auth)」「認証情報(Credentials)」を選びます。OAuthページで「新しいクライアントIDを作成(Create new Client ID)」し、アプリケーションの種類として「ウェブアプリケーション(Web application)」を選び、「同意画面を設定(Configure consent screen)」を選びます。

同意した後、「承認済みのリダイレクトURI (Authorized redirect URIs)」にアプリケーションのURL(例: http://アプリケーション名.herokuapp.com/auth/google/callback )を指定し、「クライアントIDを作成(Create Client ID)」を選びます。

[f:id:paiza:20150706124704p:plain]

「APIと認証(APIs&auth)」「API」から「Google+ API」を選び、　Google+ APIを有効にします。　
「APIと認証(APIs&auth)」「認証情報(Credentials)」で、クライアントID(Client ID, APIキー)とクライアントシークレット(Client secret, SECRETキー)を取得します。


#### ◆APIキーをHerokuに設定

APIキー・SECRETキーが取得できたら、"heroku config set"コマンドで、取得したAPIキーSECRETキーを、実行環境の環境変数として設定します。

```shell
% cd dist
% heroku config set FACEBOOK_ID=xxx
% heroku config set FACEBOOK_SECRET=xxx
% heroku config set TWITTER_ID=xxx
% heroku config set TWITTER_SECRET=xxx
% heroku config set GOOGLE_ID=xxx
% heroku config set GOOGLE_SECRET=xxx
% heroku config
% heroku restart
```


<div id="summary"></div>
■まとめ
===========
今回は、AngularJS Full-stack generatorを使ってMEANスタックのWebサービスを作成、実行、デプロイしました。MEANスタックは非常に簡単にフルスタックのWebサービスを作れるため、プロジェクトのスタートアップ・プロトタイプ作成等には特に便利です。ぜひ試してみてください。
今回はテンプレートを動かすところまでですが、次回以降では、もう少し実用的なWebサービスも作ってみたいと考えています。

[http://paiza.hatenablog.com/entry/2015/07/09/1%E6%99%82%E9%96%93%E3%81%A7Twitter%E9%A2%A8%E3%83%95%E3%83%AB%E3%82%B9%E3%82%BF%E3%83%83%E3%82%AF%E3%83%BBWeb%E3%82%B5%E3%83%BC%E3%83%93%E3%82%B9%E3%82%92%E4%BD%9C%E3%82%8B%EF%BC%81-_MEAN%E3%82%B9:embed:cite]

<div id="references"></div>
■参照
=========
AngularJS Full-Stack generator

[https://github.com/DaftMonk/generator-angular-fullstack]

<br><br>
<hr>


<a href="http://paiza.jp/">paiza</a>ではITエンジニアとしてのスキルレベル測定(9言語に対応)や、プログラミング問題による学習コンテンツ(<a href="https://paiza.jp/learning"  target="_blank">paiza Learning</a>)を提供(こちらは21言語に対応)しています。テストの結果によりS,A,B,C,D,Eの６段階でランクが分かります。自分のプログラミングスキルを客観的に知りたいという方は是非チャレンジしてみてください。

<a href="https://paiza.jp/learning"  target="_blank">http://paiza.jp
<img src="http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20140901/20140901151109.jpg"></a>
<a href="http://paiza.jp"><img src="http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20130917/20130917190908.gif?1379412762"></a>

<hr>
<br><br>

<!--/div-->