Building QA service in an hour - with MEAN stack(3)
<!--
0:1時間でフルスタックQAサービスを作る！- MEANスタック開発(3)
MEANスタックで今すぐ作る最新ウェブサービス:ジェネレータを使ってみよう編
フルスタック・ウェブ開発環境の使い方 - MEANスタックで作る最新ウェブサービス
フルスタック・Webサービスが今すぐ作れる！ - MEANスタック開発(1)
最新・最速！Webサービスが今すぐ作れる！ - MEANスタック開発(1)

1時間でQAサービスを作る！- MEANスタック開発(3)

0123456789012345678901234567890123456789012345678901234567890123
-->
<!--div class="paiza-custom-header"-->

<iframe src="https://player.vimeo.com/video/135022783?autoplay=1&loop=1&title=0&byline=0&portrait=0" width="500" height="395" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe> <p><a href="https://vimeo.com/135022783">Mean Stack 3 Demo</a> from <a href="https://vimeo.com/user34262363">paiza</a> on <a href="https://vimeo.com">Vimeo</a>.</p>

[f:id:paiza:20140712194904j:plain]( by Yoshioka Tsuneo ([twitter:@yoshiokatsuneo]))

MEAN stack(*) is a all-in-one JavaScript based web service development environment supporting front-end, back-end, and database development. A MEAN stack envinronment, AngularJS Full-Stack generator, provides the best practice to develop clean software in a timely manner.
(*) MEAN stack packs MongoDB, Express, AngularJS, and Node.js.

In the first article, I introduced how to install the MEAN stack. In the second article, I introduced how to build Twitter-like web service.

In this article, as a example of more practile web service, I introduce how to build a QA service, like Stack Overflow, Qiita, or even Reddit or HackerNews. It can be applied for features like Blog or SNS comments where users can comment, discuss, or communicate each other.

In the second article(Twitter-like service), we build a service with one page. In this QA service, we build multilpe pages using generator, input validation using validator.

The QA service has the following features:

* Listing questions
* Create, edit, or delete questions
* Create, edit, or delete answers for each question
* Create, edit, or delete comments for each question or answer.
* Tags for each question
* Markdown editting
* Searching questions, answers, or comments.
* Staring questions, answers, or comments.
* Listing all questions, my questions, or starred questions.

![](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150731/20150731152603.png)
![](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150731/20150731152617.png)

The QA service is available in the following URL:
http://paizaqa.herokuapp.com

Source code is availalbe below:
https://github.com/gi-no/paizaqa

So, let's build the QA service!

Contents
=====
<!-- [TOC] -->

* [MEANスタックのインストール](#install)
* [プロジェクトの作成](#new_project)
* [ディレクトリ構成](#structure)
* [サーバ側の質問コンポーネント(モデル、API等)の作成](#generate_server_questions)
* [クライアント側の質問一覧、作成、表示コンポーネント(コントローラ・HTML等)の作成](#generate_client_questions)
* [回答の追加](#add_answers)
* [Markdown表記の対応](#markdown)
* [質問タグの追加](#tags)
* [ユーザ認証](#user_auth)
* [入力の検証](#validation)
* [時刻表示フィルタ](#fromnow_filter)
* [コメントの追加](#add_comments)
* [お気に入りの追加](#stars)
* [全ての質問、自分の質問、お気に入りの質問の一覧表示](#navbar_all_mine_stars)
* [検索](#search)
* [日本語検索](#search_japanese)
* [無限スクロール](#infinite_scroll)
* [SNS認証](#oauth)
* [デプロイ](#deploy)
* [まとめ](#summary)

<div id="install"></div>

Install MEAN stack
================
If you have not installed, install a MEAN stack, Angular Full-Stack generator.
See 
[Installation section in the first article](http://paiza.hatenablog.com/entry/2015/07/08/最新・最速！Webサービスが今すぐ作れる！_-_MEANスタッ) for installation instruction.


<div id="new_project"></div>

Generate a new project
================
At first, we generate a project from templates. Let's name the project "paizaqa" for now. We use "yo" Yeoman command to generate the project.

```shell
% mkdir paizaqa
% cd paizaqa
% yo angular-fullstack paizaqa
```

We use almost default settings but enable SNS authentication(oAuth).

```
- Would you like to include additional oAuth strategies? 
 ◉ Google
 ◉ Facebook
❯◉ Twitter
```

Update npm packages because default versions are a bit old.

```shell
% sudo npm install -g npm-check-updates
% npm-check-updates -u
% npm install
```

Start the project.

```shell
% grunt serve
```

Browser will open generated project on http://localhost:9000/ .

![](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150731/20150731152623.png)

<div id="structure"></div>

Directory strucure
===================
The generated project have the following structure.


```
.
|-- bower.json                            Bower packages(Client-side libraries)
|-- package.json                          npm packages(Server-side libraries)
|
|-- client                                Client-side codes
|   |-- app
|   |   |-- app.js                        Client-side main JavaScript code
|   |   `-- main
|   |       |-- main.controller.js        Client-side controller code
|   |       |-- main.controller.spec.js   Client-side test code
|   |       |-- main.html                 HTML template file
|   |       |-- main.js                   Client-side routing configuration
|   |       `-- main.scss                 CSS file
|   |-- components
|   |   |-- navbar
|   |   |   |-- navbar.controller.js      Navbar controller
|   |   |   `-- navbar.html               Navbar HTML template file
|   |   `-- socket
|   |       `-- socket.service.js         Client-side WebSocket code
|   `-- index.html
|
`-- server                                Server-side code
    `-- api
        `-- thing
            |-- index.js                  Server-side API routing configuration
            |-- thing.controller.js       Server-side controller(API implementation)
            |-- thing.model.js            Server-side DB model
            |-- thing.socket.js           Server-side WebSocket implementation
            `-- thing.spec.js             Server-side test code
```

Client-side codes are deployed under a client directory, server-side codes are deployed under a server directory. By packaging files into a directory for each feature as a component, components become independent each other and whole the project is clean and easy to understand.

On client directory, each "app/MODULE" directory stores files for each URL routing as a component. The main files for directories are HTML files(ex: "main.html"), client-side controller(ex: "main.controller.js"). Other files are client-side routing configuration, test codes, CSS files, etc. Common features for the project are stored under "components" directory as a subdirectory.

On server directory, each "server/api/MODULE" directory stores files for each URL routing as a component. The main files for the directories are Database model, server-side controller. Other files are server-side routing configuration, server-side WebSocket implementation, test codes.

The client and the server send or receive data or events by communicating client-side controllers and server-side contorllers using JSON-based HTTP APIs. From MVC model's perspective, server see client as views, and client see server as models.


<div id="generate_server_questions"></div>

Create server-side question component(model, API, etc)
======================================

In this QA service, each question is stores as a document in a database.

Generate server-side question-related directory and files(DB mode, server-side controller, etc) using generator.

```shell
% yo angular-fullstack:endpoint question
```

When prompted endpoint, set default ("/api/questions").
The generator generate "server/api/question" directory with files such as "question.controller.js", "question.model.js", and "/api/question" API is prepared for use.

We edit database model to store question titles, question contents, and the list of answers. MongoDB can directly store arrays or associated arrays as a part of one JSON object. MongoDB itself is flexible schema and does not require pre-defined schema. But mongoose driver used in Angular Full-Stack generator provide schema as a additional feature to limit or validate fields, we define question related information as a schema.


server/api/question/question.model.js

```javascript
var QuestionSchema = new Schema({
  title: String,
  content: String,
});
```

<div id="generate_client_questions"></div>

Create client-side question listing, question creating, and question showing components(controllers, HTMLs, etc)
クライアント側の質問一覧、作成、表示コンポーネント(コントローラ・HTML等)の作成
=====================================

Now, we create files for question listing, question creating, and question showing components(controllers, HTMLs)

![](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150731/20150731152639.png)
![](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150731/20150731152631.png)
![](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150731/20150731152705.png)


#### 不要なファイルの削除
Remove needless files from the project.

```shell
% rm -r client/app/main
```

#### 雛形作成

Generate question related directories and files. Now, we create three directories(questionsIndex, questionsCreate, and questionsShow) for question listing, question creating, and question showing.


そして、質問関連のファイル、及びそれらをまとめたディレクトリgeneratorで作成します。ここでは質問一覧表示、質問作成、質問表示に対応した３つのディレクトリ(questionsIndex, questionCreate, questionsShow)を作成します。

When generator prompted URL routing as "What will the url of your route be?", type the following URLs.

* questionsIndex(Question listing): /
* questionsCreate(Question creating): /questions/create
* questionsShow(Question showing): /questions/show/:id

```shell
% yo angular-fullstack:route questionsIndex
? Where would you like to create this route? client/app/
? What will the url of your route be? /
% yo angular-fullstack:route questionsCreate
? Where would you like to create this route? client/app/
? What will the url of your route be? /questions/create
% yo angular-fullstack:route questionsShow
? Where would you like to create this route? client/app/
? What will the url of your route be? /questions/show/:id
```

Directories and files are generated. Now, we implement client-side controllers and HTMLs by editting the files.

#### Edit question listing controller
On the question listing controller, we retrieve question listing using "GET /api/questions" API.
Store the retrieved question to $scope variable so that we can refer from HTML file. Because we use "$http" service, add "$http" to the controller function parameters. The service based on the parameter variable name is assigned to the parameter.

client/app/questionsIndex/questionsIndex.controller.js

```javascript
  .controller('QuestionsIndexCtrl', function ($scope, $http) {
    $http.get('/api/questions').success(function(questions) {
      $scope.questions = questions;
    });
  });
```

#### Edit question listing HTML file
On question listing HTML file, we can refer "$scope.question" as "questions". By writting a attribute as 'ng-repeat="question in questions"', we can repeatedly output the elements for each question.
Also, we can refer "$scope" variable such as "{&#x7b;question.title&#x7b;}".

client/app/questionIndex/questionIndex.html

```html
<div ng-include="'components/navbar/navbar.html'"></div>

<header class="hero-unit" id="banner">
  <div class="container">
    <h1>paizaQA</h1>
    <p class="lead">Kick-start your next web app with Angular Fullstack</p>
    <img src="assets/images/yeoman.png" alt="I'm Yeoman">
  </div>
</header>

<div class="container">
  <br/>
  <div style="text-align: center">
    <a type="button" class="btn btn-primary" href="/questions/create">Ask Question</a>
  </div>

  <table class="table table-striped">
    <thead>
      <tr>
        <th>Question</th>
      </tr>
    </thead>
    <tbody>
      <tr ng-repeat="question in questions">
        <td>
          <a ng-href="/questions/show/{{question._id}}" style="font-size: large">{{question.title}}</a>
        </td>
      </tr>
    </tbody>
  </table>
</div>

<footer class="footer">
  <div class="container">
      <p>paizaQA</p>
  </div>
</footer>
```

client/app/questionIndex/questionIndex.scss

```css
#banner {
    border-bottom: none;
    margin-top: -20px;
}

#banner h1 {
    font-size: 60px;
    line-height: 1;
    letter-spacing: -1px;
}

.hero-unit {
    position: relative;
    padding: 30px 15px;
    color: #F5F5F5;
    text-align: center;
    text-shadow: 0 1px 0 rgba(0, 0, 0, 0.1);
    background: #4393B9;
}

.footer {
    text-align: center;
    padding: 30px 0;
    margin-top: 70px;
    border-top: 1px solid #E5E5E5;
}
```

#### Edit question crating controller
Implement "$scope.submit()" function called when questions are submitted.
The function stores the submitted question($scope.question) to server using "POST /api/questions" API.
After the submittion, move to submittion listing page using "$location.path('/questions')".


client/app/questionsCreate/questionsCreate.controller.js

```javascript
...
  .controller('QuestionsCreateCtrl', function ($scope, $http, $location) {
    $scope.submit = function() {
      $http.post('/api/questions', $scope.question).success(function(){
        $location.path('/');
      });
    };
  });
```

#### Edit question creating HTML file
On question creating HTML file, add "ng-submit" attribute to call "submit()" on submissions. Add 'ng-model="question.title"' attribute to input tag to synchronize between input tag and "question.title" variable bi-directionally.

client/app/questionsCreate/questionsCreate.html

```html
<div ng-include="'components/navbar/navbar.html'"></div>
<div class="container">
  <form name="form" ng-submit="submit()">
    <h2>Title:</h2>
    <input type="text" class="form-control" ng-model="question.title">
    <br>
    <h2>Question:</h2>
    <textarea rows=10 cols=80 ng-model="question.content"></textarea>
    <input type="submit" class="btn btn-primary" value="Post question">
  </form>
</div>
```

#### Edit question showing controller
On question showing controller, retrieve question contents and show it.
Question ID can be retrieved from URL("/question/show/:id") by refering ":id" part as "$stateParams.id".

client/app/questionsShow/questionsShow.controller.js

```javascript
  .controller('QuestionsShowCtrl', function ($scope, $http, $stateParams) {
    var loadQuestions = function(){
      $http.get('/api/questions/' + $stateParams.id).success(function(question) {
        $scope.question = question;
      });
    };
    loadQuestions();
  });
```

#### Edit question showing HTML file
On question showing HTML file, output question titile and contents.
We refer "$scope.question" variable set on controller as "question".


client/app/questionsShow/questionsShow.html

```html
<div ng-include="'components/navbar/navbar.html'"></div>
<div class="container" id="question-show-container">
  <div>
    <div>
      <h1>{{question.title}}</h1>
    </div>
  </div>
  <hr/>
  {{question.content}}
</div>
```

/Users/tsuneo/gino/paizaqa2/client/app/questionsShow/questionsShow.scss

```css
#question-show-container .comment{
  hr {
    margin: 0;
  };
  p {
    margin: 0;
  };
  margin-left: 100px;
};
```

#### Restart the service
Generally, we don't need to restart the service.
But now, because we changed the directory structure dramatically, we restart the service to make sure.

```
% grunt serve
```




<div id="add_answers"></div>

Create answer field
==============================
Now, though we are building QA service, we can only ask quetions and no one can answer it.
So, let's enable to create, display answers.

![](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150731/20150731152706.png)

#### Edit server-side DB model

Edit QuestionSchema to store answers. MongoDB can store JSON object including arrays into a document(corresponding a record in RDB).

server/api/question/question.model.js

```javascript
var QuestionSchema = new Schema({
  title: String,
  content: String,
  answers: [{
    content: String,
  }],
});
```


#### Edit server-side routing

Add answer submittion API to the URL routing.

server/api/question/index.js


```javascript
router.post('/:id/answers', controller.createAnswer);
```

#### Edit server-side controller

Implement answer submittion API. We can add a value to a array by using MongoDB's '$push' operator.

server/api/question/question.controller.js

```javascript
exports.createAnswer = function(req, res) {
  Question.update({_id: req.params.id}, {$push: {answers: req.body}}, function(err, num) {
    if(err) { return handleError(res, err); }
    if(num === 0) { return res.send(404); }
    exports.show(req, res);
  });
};
```


#### Edit client-side question showing controller

Add "$scope.submitAnswer()" function called on answer submittion. The function send the answer to server using "POST /api/questions/QUESTION-ID/answers" API. Reload whole question after the submittion.

client/app/questionsShow/questionsShow.controller.js

```javascript
    ...
    loadQuestions();
    $scope.newAnswer = {};
    $scope.submitAnswer = function() {
      $http.post('/api/questions/' + $stateParams.id + '/answers', $scope.newAnswer).success(function(){
        loadQuestions();
        $scope.newAnswer = {};
      });
    };
```

#### Edit client-side question showing HTML
Output the list of answer stored in the question.
Add "ng-submit" attribute to call "$scope.submitAnswer()" on answer submittion.

client/app/questionsShow/questionsShow.html

```html
  ...
  {{question.content}}
  &nbsp;
  <h3>{{question.answers.length}} Answers</h3>
  <div ng-repeat="answer in question.answers">
    <hr/>
    <div class="answer">
      {{answer.content}}
    </div>
  </div>
  <hr/>
  <h3>Your answer</h3>
  <form name="answerForm" ng-submit="submitAnswer()">
    <textarea rows=10 cols=80 ng-model="newAnswer.content"></textarea>
    <input type="submit" class="btn btn-primary" value="Submit your answer">
  </form>
</div>
```

client/app/questionsShow/questionsShow.css

```css
...
#question-show-container .answer{
  margin-left: 50px;
};
```




<div id="markdown"></div>

Support Markdown
==================
For now, we can use only plain text for questions or answers.
Let's support Markdown used like Stack Overflow.

We can just add a module and edit tags to support Markdown.

![](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150731/20150731152709.png)
![](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150731/20150731152711.png)

#### Install module

Install angular-pagedown module to support Markdown. When prompted for AngularJS, choose a option to use the latest version.

```shell
% bower install angular-pagedown --save
% grunt wiredep
...
Unable to find a suitable version for angular, please choose one:
    1) angular#~1.2 which resolved to 1.2.28 and is required by angular-pagedown#0.4.3
    2) angular#>=1.2.* which resolved to 1.4.3 and is required by paizaqa
    3) angular#>= 1.0.8 which resolved to 1.4.3 and is required by angular-ui-router#0.2.15
    4) angular#>=1 which resolved to 1.4.3 and is required by angular-bootstrap#0.11.2
    5) angular#^1.2.6 which resolved to 1.4.3 and is required by angular-socket-io#0.6.1
    6) angular#1.4.3 which resolved to 1.4.3 and is required by angular-resource#1.4.3

Prefix the choice with ! to persist it to bower.json

? Answer: 2
```

Add lines to "bower.json" to load depending files.

```javascript
{
  ...
  },
  "overrides": {
    "pagedown": {
      "main": [
        "Markdown.Converter.js",
        "Markdown.Sanitizer.js",
        "Markdown.Extra.js",
        "Markdown.Editor.js",
        "wmd-buttons.png"
      ]
    }
  }
}
```

Load depending files.

```shell
% grunt wiredep
```

Add the module to test(Karma) libraries.

karma.conf.js

```javascript
    files: [
      ...
      'client/bower_components/angular-pagedown/angular-pagedown.js',
      ...
    ]
```


#### Add module to application

To make use of the module, add "ui.pagedown" to application depending modules.

client/app/app.js

```javascript
angular.module('paizaqaApp', [
  ... ,
  'ui.pagedown',
])
```

#### Use pagedown tag

Enable Markdown input. Change from "textarea" tag to "pagdown-editor" tag and set binded variable by "content" attribute. For about output, change from "{&#x7b;&#x7d;}" to "pagedown-viewer" tag.

client/app/questionsCreate/questionsCreate.html

```html
<!-- <textarea ... ng-model="question.content">...</textarea> -->
<pagedown-editor content="question.content"></pagedown-editor>
```

client/app/questionsShow/questionsShow.html

```
<!-- {{question.content}} -->
<pagedown-viewer content="question.content"></pagedown-viewer>
...
<!-- {{answer.content}} -->
<pagedown-viewer content="answer.content"></pagedown-viewer>
...
<!-- <textarea ... ng-model="newAnswer.content"></textarea> -->
<pagedown-editor content="newAnswer.content"></pagedown-editor>
```

<div id="tags"></div>

Add question tags
==================
To make it easy to understand kinds of questions, let's add tags related to questions(ex: "Android", "Objective-C") to each question.

![](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150731/20150731152715.png)
![](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150731/20150731152716.png)

#### Edit database model
Edit QuestionSchema to store tags as a array.

```javascript
var QuestionSchema = new Schema({
  title: String,
  content: String,
  answers: [{
    content: String,
  }],
  tags: [{
    text: String,
  }],
});
```

#### Add tag editting module
Install ngTagsInput module to make it easy to edit or show tags.

```shell
% bower install ng-tags-input --save
% grunt wiredep
```

Also, add the module to test(Karma) libraries.

karma.conf.js

```javascript
    files: [
      ...
      'client/bower_components/ng-tags-input/ng-tags-input.min.js',
      ...
    ]
```

#### Add module to application
To make use of ngTagsInput module, add the module to application depending modules.

client/app/app.js

```javascript
angular.module('paizaqaApp', [
  ...
  'ngTagsInput',
])
```

#### Edit question creating HTML file
Add question tags input field using "tags-input" tag.
You can also add auto completion by adding "auto-complete" element inside "tags-input" element, but not for this time.

client/questionsCreate/questionsCreate.html

```html
    <pagedown-editor content="question.content"></pagedown-editor>
    <h2>Tags:</h2>
    <tags-input ng-model="question.tags">
      <!-- <auto-complete source="loadTags($query)"></auto-complete> -->
    </tags-input>
```

#### Edit question listing HTML file
Add tags below the question title.

client/app/questionsIndex/questionsIndex.html

```html
          <a ng-href="/questions/show/{{question._id}}" style="font-size: large">{{question.title}}</a>
          <div>
            <span ng-repeat="tag in question.tags">
              <span class="label label-info">
                {{tag.text}}
              </span>
              &nbsp;
            </span>
          </div>
```

#### Edit question showing HTML file
Add tags below the question title.

client/app/questionsShow/questionsShow.html

```html
      <h1>{{question.title}}</h1>
      <span ng-repeat="tag in question.tags">
        <span class="label label-info">
          {{tag.text}}
        </span>
      </span>
```

<div id="user_auth"></div>

User authentication
==================
For now, we don't know who submit questions and answers. So, let's add user authentication feature.
Only submitted user can edit or remove the articles. Also, store the submission date.

![](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150731/20150731152718.png)

#### Edit server-side database model

Store submitted user's object ID to questions and answers. Specify object's referring model as "ref: 'User'" so that "populate()" function can expands user IDs to User objects.

server/api/question/question.model.js

```javascript
var QuestionSchema = new Schema({
  title: String,
  content: String,
  answers: [{
    content: String,
    user: {
      type: Schema.ObjectId,
      ref: 'User'
    },
    createdAt: {
      type: Date,
      default: Date.now,
    },
  }],
  tags: [{
    text: String,
  }],
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


#### Edit server-side API routing

On the server-side API routing, add "auth.isAuthenticated()" as a Express middleware to the URL resource requiring authentication so that server-side controller can refer current login user as "req.user". 
Also, add a answer deleting API(DELETE /:id/answers/:answerId).

server/api/question/index.js

```javascript
var auth = require('../../auth/auth.service');

router.get('/', controller.index);
router.get('/:id', controller.show);
router.post('/', auth.isAuthenticated(), controller.create);
router.put('/:id', auth.isAuthenticated(), controller.update);
router.patch('/:id', auth.isAuthenticated(), controller.update);
router.delete('/:id', auth.isAuthenticated(), controller.destroy);

router.post('/:id/answers', auth.isAuthenticated(), controller.createAnswer);
router.put('/:id/answers/:answerId', auth.isAuthenticated(), controller.updateAnswer);
router.delete('/:id/answers/:answerId', auth.isAuthenticated(), controller.destroyAnswer);
```

#### Edit server-side controller
On question listing API, call "pupulate()" to expand user ID to user object.
"populate('user', 'name')" expands name field of each user object.
Also, change to return last 20 questions.
"sort({createdAt: -1})" sort by created time in descending order and "limit(20)" returns the first 20 objects.
After creating the query, call "exec()" to execute the query and receive the result in the callback function.

server/api/question/question.controller.js

```javascript
exports.index = function(req, res) {
  Question.find().sort({createdAt: -1}).limit(20).populate('user', 'name').exec(function (err, questions) {
    ...
```

Change question showing API to expand user ID to user object.

server/api/question/question.controller.js

```javascript
exports.show = function(req, res) {
  Question.findById(req.params.id).populate('user', 'name').exec(function (err, question) {
    ...
```

Change question creating API to save user as a part of question.

```javascript
exports.create = function(req, res) {
  req.body.user = req.user;
  Question.create(req.body, ...
```

On question updating and destroying API, veryfy that current login user ID is the same as the question's user ID so that
only submitted user can edit or delete the articles.

```javascript
exports.update = function(req, res) {
    ...
    if(!question) { return res.status(404).send('Not Found'); }
    if(question.user.toString() !== req.user._id.toString()){ return res.send(403); }
    ...
};
...
exports.destroy = function(req, res) {
    ...
    if(!question) { return res.status(404).send('Not Found'); }
    if(question.user.toString() !== req.user._id.toString()){ return res.send(403); }
    ...
};
```

Implement answer destroying API. Delete the answer specified by answer's ID and answer's user ID using MongoDB '$pull' operator.

```javascript
exports.destroyAnswer = function(req, res) {
  Question.update({_id: req.params.id}, {$pull: {answers: {_id: req.params.answerId , 'user': req.user._id}}}, function(err, num) {
    if(err) { return handleError(res, err); }
    if(num === 0) { return res.send(404); }
    exports.show(req, res);
  });
};
```

Implement answer updating API. On the MongoDB query, we can refer the matching index of the array(answer array in this case) as '$'.
Add a condition to match current login user and answer's user ID so that only submitted user can update the answer.

```javascript
exports.updateAnswer = function(req, res) {
  Question.update({_id: req.params.id, 'answers._id': req.params.answerId}, {'answers.$.content': req.body.content, 'answers.$.user': req.user.id}, function(err, num){
    if(err) { return handleError(res, err); }
    if(num === 0) { return res.send(404); }
    exports.show(req, res);
  });
};
```

#### Edit client-side question listing HTML

Add user name and created time to the quesion.

client/app/questionsIndex/questionsIndex.html

```html
          <a ng-href="/questions/show/{{question._id}}" style="font-size: large">{{question.title}}</a>
          <div class="clearfix"></div>
          <div style="float: right;">
            by {{question.user.name}}
             · {{question.createdAt}}
          </div>
```

#### Edit client-side question showing controller
Implement question and answer deletion function("deleteQuestion()", "deleteAnswer()") and updating function("updateQuestion()", "updateAnswer()"). Deleting functions call "DELETE /api/questions/:id" and "DELETE /api/questions/:id" API.
Updating functions call "PUT /api/questions/:id" and "PUT /api/questions/:id/answers/:id" API.

Also, implement "isOwner()" function which checks whether the user of question or answer matches current login user or not. We can use "Auth.isLoggedIn()" to check whether the user is logged in or not, and "Auth.getCurrentUser()._id" to get current login user ID.
To user "$location" and "Auth" modules, add "$location" and "Auth" parameter to the controller function. AngularJS assign the services to the parameters from the parameter names, automcatically.

client/app/questionsShow/questionsShow.controller.js

```javascript
angular.module('paizaqaApp')
  .controller('QuestionsShowCtrl', function ($scope, $http, $stateParams, Auth, $location) {
    ...
    $scope.deleteQuestion = function() {
      $http.delete('/api/questions/' + $stateParams.id).success(function(){
        $location.path('/');
      });
    };
    $scope.deleteAnswer = function(answer) {
      $http.delete('/api/questions/' + $stateParams.id + '/answers/' + answer._id).success(function(){
        loadQuestions();
      });
    };  
    $scope.updateQuestion = function() {
      $http.put('/api/questions/' + $stateParams.id, $scope.question).success(function(){
        loadQuestions();
      });
    };
    $scope.updateAnswer = function(answer) {
      $http.put('/api/questions/' + $stateParams.id + '/answers/' + answer._id, answer).success(function(){
        loadQuestions();
      });
    };
    $scope.isOwner = function(obj){
      return Auth.isLoggedIn() && obj && obj.user && obj.user._id === Auth.getCurrentUser()._id;
    };
```

#### Edit client-side question showing HTML file

Add submitted user name and created time to the question and answers.

client/app/questionsShow/questionsShow.html

```html
  <pagedown-viewer content="question.content"></pagedown-viewer>
  <div class="text-right">by {{question.user.name}} · {{question.createdAt}}</div>
...
    <pagedown-viewer content="answer.content"></pagedown-viewer>
    <div class="text-right">by {{answer.user.name}} · {{answer.createdAt}}</div>
```

Also, add delete button if current login user is the same as question or answer user.
Use "isOwner()" function to check user, and set "ng-click" attribute to call deleteQuestion/deleteAnswer() on deletion.

client/app/questionsShow/questionsShow.html

```html
    <button ng-if="isOwner(question)" type="button" class="close" ng-click="deleteQuestion()">&times;</button>
    <div>
      <h1>{{question.title}}</h1>
...
    <button ng-if="isOwner(answer)" type="button" class="close" ng-click="deleteAnswer(answer)">&times;</button>
    <pagedown-viewer content="answer.content"></pagedown-viewer>

```

Also, add editting button. "editting" varialbe is true while editting. Use "ng-show"/"ng-if" to switch element by "editting" variable. Although we are using the same "editting" variable for question and answers, answers are under a element with "ng-repeat" attribute and ng-repeat create seperate scope for each items. So, each "editting" actually refers different variables for question and each answers.

client/app/questionsShow/questionsShow.html

```html
      <h1>
        <div ng-if="! editting">{{question.title}}</div>
        <input type=text ng-model="question.title" ng-if=" editting">
      </h1>
      ...
  <pagedown-viewer content="question.content" ng-if="!editting"></pagedown-viewer>
  <pagedown-editor content="question.content" ng-if=" editting"></pagedown-editor>
  <button type="submit" class="btn btn-primary" ng-click="editting=false;updateQuestion()" ng-show=" editting">Save</button>
  <a ng-click="editting=!editting;" ng-show="isOwner(question) && !editting">Edit</a>
  ...
    <div class="answer">
      <pagedown-viewer content="answer.content" ng-if="!editting"></pagedown-viewer>
      <pagedown-editor content="answer.content" ng-if=" editting"></pagedown-editor>
      <button type="submit" class="btn btn-primary" ng-click="editting=false;updateAnswer(answer)" ng-show=" editting">Save</button>
      <a ng-click="editting=!editting;" ng-show="isOwner(answer) && !editting">Edit</a>
    </div>
    ...
```

#### Edit client-side question creating controller

If not authenticated, move to login page.

client/app/questionsCreate/questionsCreate.controller.js

```javascript
  .controller('QuestionsCreateCtrl', function ($scope, $http, $location, Auth) {
    if(! Auth.isLoggedIn()){
      $location.path('/login');
      return;
    }
    ...
```


<div id="validation"></div>

Input validation
==============
For now, we can submit forms with empty question, empty answer. Let's validate inputs not to submit without filling the field.

![valication](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150731/20150731160812.gif)

#### Install module
Install ngMessage module for the validation.

```shell
% bower install angular-messages --save
% grunt wiredep
```

Also, add the ngMessage module to test(Karma) libraries.

karma.conf.js

```javascript
    files: [
      ...
      'client/bower_components/angular-messages/angular-messages.js',
      ...
    ]
```


#### Add module to application
Add the ngMessage module to the application depending modules to use the module.

client/app/app.js

```javascript
angular.module('paizaqaApp', [
  ...
  'ngMessages',
])
```

#### On client-side HTML file, add validations to input fields
Add validations(ex: "required") to input fields. Also, to refer the validation results, add name(with "name" attribute) to the form and add name(with "name" attribute) and model(with "ng-model" attribute) to each input field. The validation result is stored as "FORM-NAME.FIELD-NAME.$error", and output the result using ng-messages/ng-message attributes in which only matching elements are shown. The whole form validation result is stored as "FORM-NAME.$invalid" and we can disable submit button when invalid.

client/app/questionsCreate/questionsCreate.html

```html
  <form name="form">
    <h2>Title:</h2>
    <input type="text" class="form-control" ng-model="question.title" name="question_title" required>
    <span class="text-danger" ng-messages="form.question_title.$error">
      <span ng-message="required">Required</span>
    </span>
    <span class="text-success" ng-show="form.question_title.$valid">OK</span>
    <br>
    <h2>Question:</h2>
    <pagedown-editor content="question.content" ng-model="question.content" name="question_content" required></pagedown-editor>
    <span class="text-danger" ng-messages="form.question_content.$error">
      <span ng-message="required">Required</span>
    </span>
    <span class="text-success" ng-show="form.question_content.$valid">OK</span>
    ...
    <input type="submit" class="btn btn-primary" ng-disabled="form.$invalid" value="Post question">
  </form>
```

client/app/questionsCreate/questionsShow.html

```html
    <pagedown-editor content="newAnswer.content" ng-model="newAnswer.content" name="answerEditor" required></pagedown-editor>
    <input type="submit" class="btn btn-primary" ng-disabled="answerForm.$invalid">Submit your answer</input>
```


<div id="fromnow_filter"></div>

Time formatting filter
============
For now, the time format for created time is UTC. Let's change it to show the time from now like Stack Overflow.

![](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150731/20150731152721.png)

#### Install Moment.js library
Install Moment.js library for time formatting.

```shell
% bower install --save momentjs
% grunt wiredep
```

Add a "moment-with-locales.min.js" file for I18N. (no need for English)

client/index.html

```html
<!-- build:js({client,node_modules}) app/vendor.js -->
  <!-- bower:js -->
  ...
  <!-- endbower -->
  <script src="bower_components/momentjs/min/moment-with-locales.min.js"></script>
  <script src="socket.io-client/socket.io.js"></script>
<!-- endbuild -->
```

#### Generate filter

Generate a filter boilerplate.

```shell
% yo angular-fullstack:filter fromNow
% grunt injector
```

#### Implement filter

Format time using Moment.js's fromNow() function. You can optionally use locale() function to set language.

client/app/fromNow/fromNow.filter.js

```javascript
    return function (input) {
      return moment(input).locale(window.navigator.language).fromNow();
    };
```

#### Use filter

Change time format from UTC to time from now, using the fromNow filter we created. To use filter, change from "{&#x7b;EXPRESSION&#x7d;}" to "{&#x7b;EXPRESSION|FILTER&#x7d;}". In this case, we change from "{&#x7b;EXPRESSION&#x7d;}"を"{&#x7b;EXPRESSION|fromNow&#x7d;}".

client/app/questionsIndex/questionsIndex.html

```html
<!-- Old: {{question.createdAt}} -->
{{question.createdAt|fromNow}}

```

client/app/questionsCreate/questionsCreate.html

```html
<!-- Old: {{question.createdAt}} -->
{{question.createdAt|fromNow}}
```

```html
<!-- Old: {{answer.createdAt}} -->
{{answer.createdAt|fromNow}}

```

#### Change test

Fix the failing test.

To load Moment.js on test, add "moment.js" to karma.conf.js

karma.conf.js

```javascript
    files: [
        ...
      'client/bower_components/momentjs/moment.js',
      ...
    ],
```

Change test code to test that the fromNow filter with current time(Date.now()) returns 'a few seconds ago'.

client/app/fromNow/fromNow.filter.spec.js

```javascript
  it('return "a few seconds ago" for now', function () {
    expect(fromNow(Date.now())).toBe('a few seconds ago');
  });
```

<div id="add_comments"></div>

Add comments
================
Now, let's add comment fields for questions and answers.

![](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150731/20150731152722.png)

#### Edit server-side database model
On QuestionSchema, store comments as a array inside the each question and answer. Each comment hold created time and submitted user.


server/api/question/question.model.js

```javascript
var QuestionSchema = new Schema({
  ...
  answers: [{
    ...
    comments: [{
      content: String,
      user: {
        type: Schema.ObjectId,
        ref: 'User'
      },
      createdAt: {
        type: Date,
        default: Date.now,
      }
    }],
  }],
  ...
  comments: [{
    content: String,
    user: {
      type: Schema.ObjectId,
      ref: 'User'
    },
    createdAt: {
      type: Date,
      default: Date.now,
    }
  }],
});
```

#### Edit server-side routing
Add the following APIs for create, update, and delete a comment to a question or an answer.

* POST /:id/comments Create a comment for a question
* PUT /:id/comments/:commentId Update a comment for a question
* DELETE /:id/comments/:commentId Delete a comment for a question
* POST /:id/answers/:answerId/comments Create a comment for a answer
* PUT /:id/answers/:answerId/comments/:commentId Update a comment for a answer
* DELETE /:id/answers/:answerId/comments/:commentId Delete a comment for a answer

/Users/tsuneo/gino/paizaqa2/server/api/question/index.js

```javascript
router.post('/:id/comments', auth.isAuthenticated(), controller.createComment);
router.put('/:id/comments/:commentId', auth.isAuthenticated(), controller.updateComment);
router.delete('/:id/comments/:commentId', auth.isAuthenticated(), controller.destroyComment);

router.post('/:id/answers/:answerId/comments', auth.isAuthenticated(), controller.createAnswerComment);
router.put('/:id/answers/:answerId/comments/:commentId', auth.isAuthenticated(), controller.updateAnswerComment);
router.delete('/:id/answers/:answerId/comments/:commentId', auth.isAuthenticated(), controller.destroyAnswerComment);
```


#### Edit server-side controller

On question listing API, expand a user ID of a comment to a user object using populate().

server/api/question/question.controller.js

```javascript
exports.show = function(req, res) {
  Question.findById(req.params.id).populate('user', 'name').populate('comments.user', 'name').populate('answers.user', 'name').populate('answers.comments.user', 'name').exec(function (err, question) {
    ...
```

Implement APIs to create, update, or delete a comment. To create a comment, add a comment to the comment array of the question document using '$push' operator. To delete a comment, delete a comment from the comment array of the question document. To update a comment, specify the updating index of the comment array of the question document using '$'. To update a comment for a answer, because we update a item inside a array of a array and only one '$' can be used to specify index, we need to iterate each item for the array.


server/api/question/question.controller.js

```javascript
/* comments APIs */
exports.createComment = function(req, res) {
  req.body.user = req.user.id;
  Question.update({_id: req.params.id}, {$push: {comments: req.body}}, function(err, num){
    if(err) {return handleError(res, err); }
    if(num === 0) { return res.send(404); }
    exports.show(req, res);
  })
}
exports.destroyComment = function(req, res) {
  Question.update({_id: req.params.id}, {$pull: {comments: {_id: req.params.commentId , 'user': req.user._id}}}, function(err, num) {
    if(err) { return handleError(res, err); }
    if(num === 0) { return res.send(404); }
    exports.show(req, res);
  });
};
exports.updateComment = function(req, res) {
  Question.update({_id: req.params.id, 'comments._id': req.params.commentId}, {'comments.$.content': req.body.content, 'comments.$.user': req.user.id}, function(err, num){
    if(err) { return handleError(res, err); }
    if(num === 0) { return res.send(404); }
    exports.show(req, res);
  });
};

/* answersComments APIs */
exports.createAnswerComment = function(req, res) {
  req.body.user = req.user.id;
  Question.update({_id: req.params.id, 'answers._id': req.params.answerId}, {$push: {'answers.$.comments': req.body}}, function(err, num){
    if(err) {return handleError(res, err); }
    if(num === 0) { return res.send(404); }
    exports.show(req, res);
  })
}
exports.destroyAnswerComment = function(req, res) {
  Question.update({_id: req.params.id, 'answers._id': req.params.answerId}, {$pull: {'answers.$.comments': {_id: req.params.commentId , 'user': req.user._id}}}, function(err, num) {
    if(err) { return handleError(res, err); }
    if(num === 0) { return res.send(404); }
    exports.show(req, res);
  });
};
exports.updateAnswerComment = function(req, res) {
  Question.find({_id: req.params.id}).exec(function(err, questions){
    if(err) { return handleError(res, err); }
    if(questions.length === 0) { return res.send(404); }
    var question = questions[0];
    var found = false;
    for(var i=0; i < question.answers.length; i++){
      if(question.answers[i]._id.toString() === req.params.answerId){
        found = true;
        var conditions = {};
        conditions._id = req.params.id;
        conditions['answers.' + i + '.comments._id'] = req.params.commentId;
        conditions['answers.' + i + '.comments.user'] = req.user._id;
        var doc = {};
        doc['answers.' + i + '.comments.$.content'] = req.body.content;
        /*jshint -W083 */
        Question.update(conditions, doc, function(err, num){
          if(err) { return handleError(res, err); }
          if(num === 0) { return res.send(404); }
          exports.show(req, res);
          return;
        });
      }
    }
    if(!found){
      return res.send(404);
    }
  });
};
```


#### Edit client-side controller

Add functions to add, update, or delete a comment that send the request to the server.

client/app/questionsShow/questionsShow.controller.js

```javascript
    $scope.submitComment = function() {
      $http.post('/api/questions/' + $stateParams.id + '/comments', $scope.newComment).success(function(){
        loadQuestions();
        $scope.newComment = {};
        $scope.editNewComment = false;
      });
    };
    $scope.submitAnswerComment = function(answer) {
      $http.post('/api/questions/' + $stateParams.id + '/answers/' + answer._id + '/comments', answer.newAnswerComment).success(function(){
        loadQuestions();
      });
    };
    $scope.deleteComment = function(comment) {
      $http.delete('/api/questions/' + $stateParams.id + '/comments/' + comment._id).success(function(){
        loadQuestions();
      });
    };
    $scope.deleteAnswerComment = function(answer, answerComment) {
      $http.delete('/api/questions/' + $stateParams.id + '/answers/' + answer._id + '/comments/' + answerComment._id).success(function(){
        loadQuestions();
      });
    };
    $scope.updateComment = function(comment) {
      $http.put('/api/questions/' + $stateParams.id + '/comments/' + comment._id, comment).success(function(){
        loadQuestions();
      });
    };
    $scope.updateAnswerComment = function(answer, answerComment) {
      $http.put('/api/questions/' + $stateParams.id + '/answers/' + answer._id + '/comments/' + answerComment._id, answerComment).success(function(){
        loadQuestions();
      });
    };
```

#### Edit client-side question showing HTML file

Output comments' content, created time, and user for the question or the answers in the question object. Also, add forms to submit new comments.


client/app/questionsShow/questionsShow.html

```html
  <div class="text-right">by <a ng-href="/users/{{question.user._id}}">{{question.user.name}}</a> · {{question.createdAt|fromNow}}</div>
  &nbsp;
  <div class="comment">
    <div ng-repeat="comment in question.comments">
      <hr/>
      <button ng-if="isOwner(comment)" type="button" class="close" ng-click="deleteComment(comment)">&times;</button>

      <pagedown-viewer content="comment.content" ng-if="!editting"></pagedown-viewer>
      <pagedown-editor content="comment.content" ng-if=" editting"></pagedown-editor>
      <button type="submit" class="btn btn-primary" ng-click="editting=false;updateComment(comment)" ng-show=" editting">Save</button>
      <a ng-click="editting=!editting;" ng-show="isOwner(comment) && !editting">Edit</a>

      <div class="text-right" style="vertical-align: bottom;">by <a ng-href="/users/{{comment.user._id}}">{{comment.user.name}}</a> · {{comment.createdAt|fromNow}}</div>
      <div class="clearfix"></div>
    </div>
    <hr/>
    <a ng-click="editNewComment=!editNewComment;">add a comment</a>
    <form ng-if="editNewComment" name="commentForm">
      <pagedown-editor content="newComment.content" editor-class="'comment-wmd-input'"
        ng-model="newComment.content" name="commentEditor" required>
      </pagedown-editor>
      <button type="button" class="btn btn-primary" ng-click="submitComment(questionComment)" ng-disabled="commentForm.$invalid">Add Comment</button>
    </form>
  </div>
  ...
  <h3>{{question.answers.length}} Answers</h3>
  <div ng-repeat="answer in question.answers">
    ...
    <div class="text-right">by {{answer.user.name}} · {{answer.createdAt|fromNow}}</div>
    <div class="comment">
      <div ng-repeat="comment in answer.comments">
        <hr/>
        <button ng-if="isOwner(comment)" type="button" class="close" ng-click="deleteAnswerComment(answer, comment)">&times;</button>
 
        <pagedown-viewer content="comment.content" ng-if="!editting"></pagedown-viewer>
        <pagedown-editor content="comment.content" ng-if=" editting"></pagedown-editor>
        <button type="submit" class="btn btn-primary" ng-click="editting=false;updateAnswerComment(answer, comment)" ng-show=" editting">Save</button>
        <a ng-click="editting=!editting;" ng-show="isOwner(comment) && !editting">Edit</a>

        <div class="text-right">by <a ng-href="/users/{{question.user._id}}">{{comment.user.name}}</a> · {{comment.createdAt|fromNow}}</div>
        <div class="clearfix"></div>
      </div>
      <hr/>
      <a ng-click="editNewAnswerComment=!editNewAnswerComment;answer.newAnswerComment={}">add a comment</a>
      <form ng-if="editNewAnswerComment" name="answer_{{answer.id}}_comment">
        <hr/>
        <pagedown-editor content="answer.newAnswerComment.content" editor-class="'comment-wmd-input'"
          ng-model="answer.newAnswerComment.content" required>
        </pagedown-editor>
        <button type="button" class="btn btn-primary" ng-click="submitAnswerComment(answer)" ng-disabled="answer_{{answer.id}}_comment.$invalid">Add Comment</button>
      </form>
    </div>
  </div>
  <hr/>
  <h3>Your answer</h3>
  ...
```

/Users/tsuneo/gino/paizaqa2/client/app/questionsShow/questionsShow.scss

```css
.comment-wmd-input {
  height: 50px;
  width: 100%;
  background-color: Gainsboro;
  border: 1px solid DarkGray;
};
```

<div id="stars"></div>

Adding stars
==============
Adding a feature to star or unstar questions, answers, and comments for questions or answers.

![](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150731/20150731152723.png)


#### Edit server-side database model

Store the list of starring users for a question, a answer, and a comment for a question or answer to a "stars" field of a question document as an array.

server/api/question/question.model.js

```javascript
var QuestionSchema = new Schema({
  ...
  answers: [{
    ...
    comments: [{
      ...
      stars: [{
        type: Schema.ObjectId,
        ref: 'User'
      }],
      ...
    }],
    stars: [{
      type: Schema.ObjectId,
      ref: 'User'
    }],
    ...
  }],
  ...
  comments: [{
    ...
    stars: [{
      type: Schema.ObjectId,
      ref: 'User'
    }],
    ...
  }],
  stars: [{
    type: Schema.ObjectId,
    ref: 'User'
  }],
  ...
});
```

#### Add server-side routing

Add the following APIs to star or unstar a question, a answer, or a comment for question or answer.

* POST /:id/star Star a question
* DELETE /:id/star Unstar a question
* POST /:id/answers/:answerId/star Star a answer
* DELETE /:id/answers/:answerId/unstar Unstar a answer
* POST /:id/comments/:commentId/star Star a comment for a question
* DELETE /:id/comments/:commentId/unstar Unstar a comment for a question
* POST /:id/answers/:answerId/comments/:commentId/star Star a comment for a answer
* DELETE /:id/answers/:answerId/comments/:commentId/star Unstar a comment for a answer

Because these APIs are related to user, register "auth.isAuthenticate()" to enable authentication.

server/api/question/index.js

```javascript
router.put('/:id/star', auth.isAuthenticated(), controller.star);
router.delete('/:id/star', auth.isAuthenticated(), controller.unstar);
router.put('/:id/answers/:answerId/star', auth.isAuthenticated(), controller.starAnswer);
router.delete('/:id/answers/:answerId/star', auth.isAuthenticated(), controller.unstarAnswer);
router.put('/:id/comments/:commentId/star', auth.isAuthenticated(), controller.starComment);
router.delete('/:id/comments/:commentId/star', auth.isAuthenticated(), controller.unstarComment);
router.put('/:id/answers/:answerId/comments/:commentId/star', auth.isAuthenticated(), controller.starAnswerComment);
router.delete('/:id/answers/:answerId/comments/:commentId/star', auth.isAuthenticated(), controller.unstarAnswerComment);
```



#### Edit server-side database model

On database model, for questions, answers, and comments for questions or answers, store the list of starred user as a array.
On "update()" function, we can refer the matched index of a array using "$". For the list of a starred user of a comment array of a answer array, because we can only use one "$", we need to iterate the array explicitly.

server/api/question/question.controller.js

```javascript
exports.star = function(req, res) {
  Question.update({_id: req.params.id}, {$push: {stars: req.user.id}}, function(err, num){
    if(err) { return handleError(res, err); }
    if(num === 0) { return res.send(404); }
    exports.show(req, res);
  });
};
exports.unstar = function(req, res) {
  Question.update({_id: req.params.id}, {$pull: {stars: req.user.id}}, function(err, num){
    if(err) { return handleError(res, err); }
    if(num === 0) { return res.send(404); }
    exports.show(req, res);
  });
};
...
exports.starAnswer = function(req, res) {
  Question.update({_id: req.params.id, 'answers._id': req.params.answerId}, {$push: {'answers.$.stars': req.user.id}}, function(err, num){
    if(err) { return handleError(res, err); }
    if(num === 0) { return res.send(404); }
    exports.show(req, res);
  });
};
exports.unstarAnswer = function(req, res) {
  Question.update({_id: req.params.id, 'answers._id': req.params.answerId}, {$pull: {'answers.$.stars': req.user.id}}, function(err, num){
    if(err) { return handleError(res, err); }
    if(num === 0) { return res.send(404); }
    exports.show(req, res);
  });
};
...
exports.starComment = function(req, res) {
  Question.update({_id: req.params.id, 'comments._id': req.params.commentId}, {$push: {'comments.$.stars': req.user.id}}, function(err, num){
    if(err) { return handleError(res, err); }
    if(num === 0) { return res.send(404); }
    exports.show(req, res);
  });
};
exports.unstarComment = function(req, res) {
  Question.update({_id: req.params.id, 'comments._id': req.params.commentId}, {$pull: {'comments.$.stars': req.user.id}}, function(err, num){
    if(err) { return handleError(res, err); }
    if(num === 0) { return res.send(404); }
    exports.show(req, res);
  });
};
...
var pushOrPullStarAnswerComment = function(op, req, res) {
  Question.find({_id: req.params.id}).exec(function(err, questions){
    if(err) { return handleError(res, err); }
    if(questions.length === 0) { return res.send(404); }
    var question = questions[0];
    var found = false;
    for(var i=0; i < question.answers.length; i++){
      if(question.answers[i]._id.toString() === req.params.answerId){
        found = true;
        var conditions = {};
        conditions._id = req.params.id;
        conditions['answers.' + i + '.comments._id'] = req.params.commentId;
        var doc = {};
        doc[op] = {};
        doc[op]['answers.' + i + '.comments.$.stars'] = req.user.id;
        // Question.update({_id: req.params.id, 'answers.' + i + '.comments._id': req.params.commentId}, {op: {('answers.' + i + '.comments.$.stars'): req.user.id}}, function(err, num){
        /*jshint -W083 */
        Question.update(conditions, doc, function(err, num){
          if(err) { return handleError(res, err); }
          if(num === 0) { return res.send(404); }
          exports.show(req, res);
          return;
        });
      }
    }
    if(!found){
      return res.send(404);
    }
  });
};
exports.starAnswerComment = function(req, res) {
  pushOrPullStarAnswerComment('$push', req, res);
};
exports.unstarAnswerComment = function(req, res) {
  pushOrPullStarAnswerComment('$pull', req, res);
};
```

#### Edit client-side question showing controller

Add functions to star, unstar, or check staring status for a question, a answer, or a comment for a question or a answer.
We use sub-pathname parameter to use the common function for questions, answers, or comments.

client/app/questionsShow/questionsShow.controller.js


```javascript
    $scope.isStar = function(obj){
      return Auth.isLoggedIn() && obj && obj.stars && obj.stars.indexOf(Auth.getCurrentUser()._id)!==-1;
    };
    $scope.star = function(subpath) {
      $http.put('/api/questions/' + $scope.question._id + subpath + '/star').success(function(){
        loadQuestions();
      });
    };
    $scope.unstar = function(subpath) {
      $http.delete('/api/questions/' + $scope.question._id + subpath + '/star').success(function(){
        loadQuestions();
      });
    };
```


#### Edit client-side question showing HTML file

Show the staring status as star icon. We can click the star icon to star or unstar.

client/app/questionsShow/questionsShow.html

```html
    <div style="float: left;font-size: x-large; padding: 0; width: 2em; text-align: center;">
      <button ng-if=" isStar(question)" type="button" style="background: transparent; border: 0;" ng-click="unstar('')">
        <span class="glyphicon glyphicon-star" style="color: #CF7C00;" ></span>
      </button>
      <button ng-if="!isStar(question)" type="button" style="background: transparent; border: 0;" ng-click="star('')"  >
        <span class="glyphicon glyphicon-star-empty"></span>
      </button>
      <br/>
      <div>{{question.stars.length}}</div>
    </div>
    
    <div>
      <h1>
        <div ng-if="! editting">{{question.title}}</div>
```

```html
      <div style="float: left;font-size: normal; padding: 0; width: 2em; text-align: center;">
        <button ng-if=" isStar(comment)" type="button" style="background: transparent; border: 0;" ng-click="unstar('/comments/' + comment._id)">
          <span class="glyphicon glyphicon-star" style="color: #CF7C00;" ></span>
        </button>
        <button ng-if="!isStar(comment)" type="button" style="background: transparent; border: 0;" ng-click="  star('/comments/' + comment._id)"  >
          <span class="glyphicon glyphicon-star-empty"></span>
        </button>
        <br/>
        <div>{{comment.stars.length}}</div>
      </div>

      <pagedown-viewer content="comment.content" ng-if="!editting"></pagedown-viewer>
```

```html
    <div style="float: left;font-size: large; padding: 0; width: 2em; text-align: center;">
      <button ng-if=" isStar(answer)" type="button" style="background: transparent; border: 0;" ng-click="unstar('/answers/' + answer._id)">
        <span class="glyphicon glyphicon-star" style="color: #CF7C00;" ></span>
      </button>
      <button ng-if="!isStar(answer)" type="button" style="background: transparent; border: 0;" ng-click="  star('/answers/' + answer._id)"  >
        <span class="glyphicon glyphicon-star-empty"></span>
      </button>
      <br/>
      <div>{{answer.stars.length}}</div>
    </div>

    <div class="answer">
```

```html
        <div style="float: left;font-size: normal; padding: 0; width: 2em; text-align: center;">
          <button ng-if=" isStar(comment)" type="button" style="background: transparent; border: 0;" ng-click="unstar('/answers/' + answer._id + '/comments/' + comment._id)">
            <span class="glyphicon glyphicon-star" style="color: #CF7C00;" ></span>
          </button>
          <button ng-if="!isStar(comment)" type="button" style="background: transparent; border: 0;" ng-click="  star('/answers/' + answer._id + '/comments/' + comment._id)"  >
            <span class="glyphicon glyphicon-star-empty"></span>
          </button>
          <br/>
          <div>{{comment.stars.length}}</div>
        </div>

        <pagedown-viewer content="comment.content" ng-if="!editting"></pagedown-viewer>
```

#### Edit client-side question listing controller

Add "isStar()" function to show the staring status on question listing.

client/app/questionsIndex/questionsIndex.controller.js

```javascript
  .controller('QuestionsIndexCtrl', function ($scope, $http, Auth, $location) {
    ...
    $scope.isStar = function(obj){
      return Auth.isLoggedIn() && obj && obj.stars && obj.stars.indexOf(Auth.getCurrentUser()._id)!==-1;
    };
```

#### Edit client-side question listing HTML file

On question listing, show the number of staring user for the question, and the number of answers.

client/app/questionsIndex/questionsIndex.html

```html
  <table class="table table-striped">
    <thead>
      <tr>
        <th width="20">Stars</th>
        <th width="20">Answers</th>
        <th>Question</th>
      </tr>
    </thead>
    <tbody>
      <tr ng-repeat="question in questions">
        <td style="text-align: center; vertical-align:middle">
          <div style="font-size: xx-large;">{{question.stars.length}}</div>
        </td>
        <td style="text-align: center; vertical-align:middle">
          <div style="font-size: xx-large;">{{question.answers.length}}</div>
        </td>
        <td>
          <div style="float: right;">
            <span ng-if=" isStar(question)" class="glyphicon glyphicon-star" style="color: #CF7C00;" ></span>
            <span ng-if="!isStar(question)" class="glyphicon glyphicon-star-empty"></span>
          </div>
          <a ng-href="/questions/show/{{question._id}}" style="font-size: large">{{question.title}}</a>
          <div class="clearfix"></div>
          <div style="float: right;">
            by <a ng-href="/users/{{question.user._id}}">{{question.user.name}}</a>
             · {{question.createdAt|fromNow}}
          </div>
          <div>
            <span ng-repeat="tag in question.tags">
              <span class="label label-info">
                {{tag.text}}
              </span>
              &nbsp;
            </span>
          </div>
          <div class="clearfix"></div>
        </td>
      </tr>
    </tbody>
  </table>
```

<div id="navbar_all_mine_stars"></div>

Question listing for all questions, my questions, and starred questions
===============
For now, all questions are always listed. Let's enable to choose from all questions, my questions, and starred questions. 

![](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150731/20150731152724.png)


#### Edit client-side routing

Assign the following URLs for all questions, my questions, and starred questions.

* / :All questions
* /users/:userId :My questions
* /users/:userId/starred :Starred questions

Use the same controller and HTML template and change the searching query by "query" variable.
For my quetions listing, the query checks whether the user of the question is the same as current login user or not.
For starred questions listing, the query checks whether the starred user for the question, the answers, or the comments contains current login user or not.

client/app/questionsIndex/questionsIndex.js


```javascript
...
  .config(function ($stateProvider) {
    $stateProvider
      .state('questionsIndex', {
        url: '/',
        templateUrl: 'app/questionsIndex/questionsIndex.html',
        controller: 'QuestionsIndexCtrl',
        resolve: {
          query: function(){return {};}
        },
      })
      .state('starredQuestionsIndex', {
        url: '/users/:userId/starred',
        templateUrl: 'app/questionsIndex/questionsIndex.html',
        controller: 'QuestionsIndexCtrl',
        resolve: {
          query: function($stateParams){
            return {
              $or: [
                {'stars': $stateParams.userId},
                {'answers.stars': $stateParams.userId},
                {'comments.stars': $stateParams.userId},
                {'answers.comments.stars': $stateParams.userId},
              ]
            };
          }
        },
      })
      .state('userQuestionsIndex', {
        url: '/users/:userId',
        templateUrl: 'app/questionsIndex/questionsIndex.html',
        controller: 'QuestionsIndexCtrl',
        resolve: {
          query: function($stateParams){
            return {user: $stateParams.userId};
          }
        },
      });

  });
```

#### Edit client-side question listing controller
Send the query set on routing to the server.

client/app/questionsIndex/questionsIndex.controller.js

```javascript
  .controller('QuestionsIndexCtrl', function ($scope, $http, Auth, query) {
    $http.get('/api/questions', {params: {query: query}}).success(function(questions) {
```

#### Edit client-side question listing controller test
Add empty "query" to test code not to cause error.

client/app/questionsIndex/questionsIndex.controller.spec.js

```javascript
    QuestionsIndexCtrl = $controller('QuestionsIndexCtrl', {
      $scope: scope,
      query: {},
    });
```

#### Edit server-side controller
Send the received query to database.

server/api/question/question.controller.js

```javascript
exports.index = function(req, res) {
  var query = req.query.query && JSON.parse(req.query.query);
  Question.find(query).sort(...
```

#### Edit client-side Navbar controller
On Navbar, add links for all questions, my quetions, starred questions.
Because we need to change the URL or enable/disable for links before login or logout, use functions instead of variables for those information on menu items.

client/components/navbar/navbar.controller.js

```javascript
  .controller('NavbarCtrl', function ($scope, $location, Auth, $state) {
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
    ...
```

#### Edit client-side Navbar HTML
Retrieve linked URLs using "$scope.item.link()" function.
Also, show the link only when "$scope.item.show()" returns true.


client/components/navbar/navbar.html

```html
      <ul class="nav navbar-nav">
        <li ng-repeat="item in menu" ng-class="{active: isActive(item.link())}" ng-show="item.show()">
            <a ng-href="{{item.link()}}">{{item.title}}</a>
        </li>
      </ul>
```

<div id="search"></div>

Search
============
With MongoDB's full-text search, let's add a feature to search from question title, contents, comments, or answers.
MongoDBの全文検索機能を使って、タイトル・質問内容・質問コメント・回答・回答コメントから検索できるようにしてみます。

![](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150731/20150731152725.png)

#### Edit client-side Navbar HTML file

Add serach box to Navbar. Call "search()" function on search.

```javascript
      <form class="navbar-form navbar-left" role="search" ng-submit="search(keyword)">
        <div class="input-group">
          <input type="text" class="form-control" placeholder="Search" ng-model="keyword">
          <span class="input-group-btn">
            <button type="submit" class="btn btn-default">
              <span class="glyphicon glyphicon-search">
              </span>
            </button>
          </span>
        </div>
      </form>
      <ul class="nav navbar-nav navbar-right">
```

#### Edit client-side Navbar controller

client/components/navbar/navbar.controller.js

```javascript
    $scope.search = function(keyword) {
      $state.go('questionsIndex', {keyword: keyword}, {reload: true});
    };
```

#### Edit server-side database model

To add index for full-text search for question titles, question contents, comments, and answers, use "QuestionSchema.index()" function and specify search target fields as 'text'.
Although MongoDB can automatically set index name, because MongoDB does not work as intended if the length of the name is long and exceeds 128 bytes, let's explicitly specify the index name(ex: 'question_schema_index').
(Ref: [We need to specify name explicitly for long schema.](
http://docs.mongodb.org/manual/reference/limits/#Index-Name-Length))

server/api/question/question.model.js

```javascript
QuestionSchema.index({
  'title': 'text',
  'content': 'text',
  'tags.text': 'text',
  'answers.content': 'text',
  'comments.content': 'text',
  'answers.comments.content': 'text',
}, {name: 'question_schema_index'});
```


#### Edit client-side question listing routing

To accept search keyword as "keyword" URL parameter, add "/?keyword" to "url" field of the routing information.

client/app/questionsIndex/questionsIndex.js

```javascript
      .state('questionsIndex', {
        url: '/?keyword',
        ...
```

#### Edit client-side question showing controller
Set the query using MongoDB's '$text' and '$search' parameter to search by 'keyword' parameter.


client/app/questionsIndex/questionsIndex.controller.js

```javascript
  .controller('QuestionsIndexCtrl', function ($scope, $http, $location, query) {
    ...
    var keyword = $location.search().keyword;
    if(keyword){
      query = _.merge(query, {$text: {$search: keyword}});
    }
    $http.get('/api/questions',...
```


<div id="search_japanese"></div>

日本語検索
==============
MongoDBの全文検索は、日本語検索に対応していません。日本語検索を行えるように、TinySegmenterで分かち書きを行ってみます。

![](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150731/20150731160824.gif)

#### クライアント側ライブラリTinySegmenterをインストール

TinySegmenterをインストールします。

```shell
% npm install --save r7kamura/tiny-segmenter
```

サーバ側DBモデルで、分かち書き結果を保存するフィールドをsearchTextとして作成します。

server/api/question/question.mode.js

```javascript
var QuestionSchema = new Schema({
  ...
  searchText: String,
});

```

インデックスにsearchTextフィールドを追加します。

server/api/question/question.mode.js

```javascript
QuestionSchema.index({
  ...
  'searchText': 'text',
}, {name: 'question_schema_index'});
```

分かち書きを行う関数を追加し、pre('save')を使って保存前に呼び出します。また質問オブジェクトの静的関数で更新できるようにもしておきます。

```javascript
var TinySegmenter = require('tiny-segmenter');

var getSearchText = function(question){
  var tinySegmenter = new TinySegmenter();
  var searchText = "";
  searchText += tinySegmenter.segment(question.title).join(' ') + " ";
  searchText += tinySegmenter.segment(question.content).join(' ') + " ";
  question.answers.forEach(function(answer){
    searchText += tinySegmenter.segment(answer.content).join(' ') + " ";
    answer.comments.forEach(function(comment){
      searchText += tinySegmenter.segment(comment.content).join(' ') + " ";
    });
  });
  question.comments.forEach(function(comment){
    searchText += tinySegmenter.segment(comment.content).join(' ') + " ";
  });
  console.log("searchText", searchText);
  return searchText;
};
QuestionSchema.statics.updateSearchText = function(id, cb){
  this.findOne({_id: id}).exec(function(err, question){
    if(err){ if(cb){cb(err);} return; }
    var searchText = getSearchText(question);
    this.update({_id: id}, {searchText: searchText}, function(err, num){
      if(cb){cb(err);}
    });
  }.bind(this));
};

QuestionSchema.pre('save', function(next){
  this.searchText = getSearchText(this);
  next();
});
```

pre('save')はupdate()での更新時は呼び出されませんので、サーバ側コントローラでupdate関数での更新後に分かち書きを行いsearchTextフィールドを更新するようにします。

server/api/question/question.controller.js

```javascript
exports.createAnswer = function(req, res) {
    ...
    exports.show(req, res);
    Question.updateSearchText(req.params.id);
  });
};
exports.destroyAnswer = function(req, res) {
  ...
    exports.show(req, res);
    Question.updateSearchText(req.params.id);
  });
};
exports.updateAnswer = function(req, res) {
    ...
    exports.show(req, res);
    Question.updateSearchText(req.params.id);
  });
};
...
exports.createComment = function(req, res) {
    ...
    exports.show(req, res);
    Question.updateSearchText(req.params.id);
  })
};
exports.destroyComment = function(req, res) {
    ...
    exports.show(req, res);
    Question.updateSearchText(req.params.id);
  });
};
exports.updateComment = function(req, res) {
    ...
    exports.show(req, res);
    Question.updateSearchText(req.params.id);
  });
};
...
exports.createAnswerComment = function(req, res) {
    ...
    exports.show(req, res);
    Question.updateSearchText(req.params.id);
  })
};
exports.destroyAnswerComment = function(req, res) {
    ...
    exports.show(req, res);
    Question.updateSearchText(req.params.id);
  });
};
exports.updateAnswerComment = function(req, res) {
  ...
          exports.show(req, res);
          Question.updateSearchText(req.params.id);
  ...
};
```

<div id="infinite_scroll"></div>

無限スクロール
============
現状、最新20件の質問しか表示することができていませんので、無限スクロールで過去の質問も表示できるようにしてみます。


#### クライアント側ライブラリngInfiniteScrollのインストール

無限スクロール用のライブラリngInfiniteScrollをインストールします。

```shell
% bower install --save ngInfiniteScroll
% grunt wiredep
```

テスト(Karma)で読み込むライブラリにも追加します。

karma.conf.js

```javascript
    files: [
      ...
      'client/bower_components/ngInfiniteScroll/build/ng-infinite-scroll.js',
      ...
    ]
```

#### アプリケーションの依存モジュール追加

ngInfiniteScrollをアプリケーションの依存モジュールに追加して利用できるようにします。


client/app/app.js

```javascript
angular.module('paizaqaApp', [
  ...
  'infinite-scroll',
 ])
```

#### クライアント側質問一覧コントローラの変更

スクロース時に表示できるように、nextPage関数を実装します。最後の質問IDより古いIDの質問を返すように、クエリで"{_id: {$lt: lastID}}"と条件を指定します。読み込み中状態を$scope.busyで、質問がまだあるかを$scope.noMoreDataに保持します。

client/app/questionsIndex/questionsIndex.controller.js

```javascript
    $scope.busy = true;
    $scope.noMoreData = false;

    $http.get('/api/questions', {params: {query: query}}).success(function(questions) {
      $scope.questions = questions;
      if($scope.questions.length < 20){
        $scope.noMoreData = true;
      }
      $scope.busy = false;
    });
    ...
    $scope.nextPage = function(){
      if($scope.busy){ return; }
      $scope.busy = true;
      var lastId = $scope.questions[$scope.questions.length-1]._id;
      var pageQuery = _.merge(query, {_id: {$lt: lastId}});
      $http.get('/api/questions', {params: {query: pageQuery}}).success(function(questions){
        $scope.questions = $scope.questions.concat(questions);
        $scope.busy = false;
        if(questions.length === 0){
          $scope.noMoreData = true;
        }
      });
    };
```

#### クライアント側質問一覧HTMLの変更
一番下までスクロールしたら次の質問を読み込むように、infinite-scroll属性でnextPage()関数を呼び出します。また、読み込み中やすべてを読み込み済みの場合は無限スクロールを無効にします。読み込み中は"Loading data"と表示するようにしておきます。ng-show='busy'と指定することでbusyがtrueの場合のみ要素が表示されます。

```html
<div class="container" infinite-scroll='nextPage()' infinite-scroll-disabled='busy || noMoreData'>
  ...
  <div ng-show='busy'>Loading data...</div>
</div>
```

<div id="oauth"></div>

SNS認証
=========
SNS認証(Facebook, Twitter, Google)を利用する場合、APIキーとSECRETキーの登録を行います。登録手順は[第一回の手順](http://paiza.hatenablog.com/entry/2015/07/08/最新・最速！Webサービスが今すぐ作れる！_-_MEANスタッ#sns_link)を参照ください。

また、Facebook認証については、FacebookのAPIが変わった関係で以下のようにprofileFieldsを指定する必要があります。

server/auth/facebook/passport.js

```
exports.setup = function (User, config) {
  passport.use(new FacebookStrategy({
      profileFields: ['displayName', 'name', 'profileUrl', 'id', 'email',  'photos', 'gender', 'locale', 'timezone', 'updated_time', 'verified'],
      clientID: ...
    ...
```

<div id="deploy"></div>

デプロイ
============
それではHerokuにデプロイをします。デプロイ設定は"yo angular-fullstack:heroku"コマンドで行います。また、無料プランがあるMongoDBモジュールMongoLabもインストールしておきます。

```shell
% yo angular-fullstack:heroku
% cd dist
% heroku addons:add mongolab
```

次回以降のデプロイでは、"grunt"でビルドを行い、"grunt buildcontrol:heroku"でデプロイします。

```shell
% grunt
% grunt buildcontrol:heroku
```

デプロイできたら、http://アプリケーション名.herokuapp.com/ で動作確認をします。

正しく表示できていない場合、Herokuのログを確認します。

```shell
% cd dist
% heroku logs
```

MongoDBのデータについては、MongoHubなどのGUIツールで確認すると便利です。MongoDBのURLはHerokuから取得します。

```shell
% heroku config
...
MONGOLAB_URI: mongodb://ユーザ名:パスワード@ホスト名:ポート番号/データベース名
...

```

<div id="summary"></div>

まとめ
===============
MEANスタックAngularJS Full-Stack generatorを用いてQAサイトを作成する手順を紹介しました。MEANスタックを使えば、JavaScriptのみでサーバからクライアントまで、見通しの良いサービスを簡単に作ることができます。ジェネレータを使うことで雛形となるコードを自動生成ができますので、変更を加えていくだけでサービスが作れるようになります。特に、スタートアップやプロトタイプで短時間・少ない人数で試行錯誤しながらサービスを作り上げる時には非常に便利です。

ぜひアイデアを生かして、ウェブサービスを作ってみてはいかがでしょうか？

手順通りに動作しないなど、気づいた点がありましたら、コメント等でフィードバックいただけるとうれしいです。

今後ともMEANスタックを使ったたのWebサービスの作り方も紹介していきたいと思います。


<br><br>
<hr>


<a href="http://paiza.jp/">paiza</a>ではITエンジニアとしてのスキルレベル測定(9言語に対応)や、プログラミング問題による学習コンテンツ(<a href="https://paiza.jp/learning"  target="_blank">paiza Learning</a>)を提供(こちらは21言語に対応)しています。テストの結果によりS,A,B,C,D,Eの６段階でランクが分かります。自分のプログラミングスキルを客観的に知りたいという方は是非チャレンジしてみてください。

<a href="https://paiza.jp/learning"  target="_blank">http://paiza.jp
<img src="http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20140901/20140901151109.jpg"></a>
<a href="http://paiza.jp"><img src="http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20130917/20130917190908.gif?1379412762"></a>

<hr>
<br><br>


