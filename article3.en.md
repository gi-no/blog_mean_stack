<!--
Building a QA web service in an hour - MEAN stack development(3)
0:1時間でフルスタックQAサービスを作る！- MEANスタック開発(3)
MEANスタックで今すぐ作る最新ウェブサービス:ジェネレータを使ってみよう編
フルスタック・ウェブ開発環境の使い方 - MEANスタックで作る最新ウェブサービス
フルスタック・Webサービスが今すぐ作れる！ - MEANスタック開発(1)
最新・最速！Webサービスが今すぐ作れる！ - MEANスタック開発(1)

1時間でQAサービスを作る！- MEANスタック開発(3)

0123456789012345678901234567890123456789012345678901234567890123
-->
<!--div class="paiza-custom-header"-->

<iframe src="https://player.vimeo.com/video/136183259?autoplay=1&loop=1&title=0&byline=0&portrait=0" width="500" height="350" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>

<div style="text-align:right">(Japanese article is <a href="http://paiza.hatenablog.com/entry/meanstack_howto_3">here</a>.)</div>


[f:id:paiza:20140712194904j:plain]  (by Yoshioka Tsuneo ([twitter:@yoshiokatsuneo]))

MEAN stack (*) is an all-in-one JavaScript-based web service development environment supporting front-end, back-end, and database development. A MEAN stack environment, AngularJS Full-Stack generator, provides the best practice to develop clean software quickly.

(*) MEAN stack packs MongoDB, Express, AngularJS, and Node.js.

[In the first article](http://engineering.paiza.io/entry/2015/07/08/153011), I introduced how to install the MEAN stack.
[In the second article](http://engineering.paiza.io/entry/2015/07/09/154028), I introduced how to build Twitter-like web service.

In this article, as an example of a more practical web service, I introduce how to build a QA service like Stack Overflow, Qiita, or even Reddit or HackerNews. The way can be used for features like Blog or SNS comments where users can comment, discuss, or communicate each other.



In the second article (Twitter-like service), we build a service with one page. In this QA service, we will build multiple pages using generator. We also use input validations using validators.

The QA service has the following features:

* Listing questions
* Create, edit, or delete questions
* Create, edit, or delete answers for each question
* Create, edit, or delete comments on each question or answer
* Tags for each question
* Markdown editing
* Searching questions, answers, or comments
* Staring questions, answers, or comments
* Listing all questions, my questions, or starred questions

![](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150813/20150813170623.png)
![](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150813/20150813170615.png)

The QA service is available in the following URL:
http://paizaqa.herokuapp.com

Source code is available below:
https://github.com/gi-no/paizaqa

So, let's build the QA service!

Contents
=====
<!-- [TOC] -->

* [Installing MEAN stack](#install)
* [Generating a new project](#new_project)
* [Directory structure](#structure)
* [Creating server-side question component(model, API, etc.)](#generate_server_questions)
* [Creating client-side question-listing, question-creating, and question-showing components (controllers, HTMLs, etc.)](#generate_client_questions)
* [Creating answer field](#add_answers)
* [Using Markdown](#markdown)
* [Adding question tags](#tags)
* [User authentication](#user_auth)
* [Input validation](#validation)
* [Time formatting filter](#fromnow_filter)
* [Adding comments](#add_comments)
* [Adding stars](#stars)
* [Question listing for all questions, my questions, and starred questions](#navbar_all_mine_stars)
* [Search](#search)
* [Japanese search](#search_japanese)
* [Infinite scroll](#infinite_scroll)
* [SNS authentication](#oauth)
* [Deploying](#deploy)
* [Summary](#summary)

<div id="install"></div>

Installing MEAN stack
================
If you have not installed, install a MEAN stack, Angular Full-Stack generator.
See 
[Installation section in the first article](http://paiza.hatenablog.com/entry/2015/07/08/最新・最速！Webサービスが今すぐ作れる！_-_MEANスタッ) for installation instruction.

Confirm that installed AngularJS Full-Stack generator is ver3.3.0 or later.

```shell
$ npm ls -g generator-angular-fullstack
/usr/local/lib
└── generator-angular-fullstack@3.3.0 
```

If it is older than ver3.3.0, update to the latest version.

```shell
$ sudo npm update -g generator-angular-fullstack
```

<div id="new_project"></div>

Generating a new project
================
At first, we generate a project from templates. Let's name the project "paizaqa" for now. We use "yo" Yeoman command to generate the project.

```shell
% mkdir paizaqa
% cd paizaqa
% yo angular-fullstack paizaqa
```

We use almost all default settings but enable SNS authentication (oAuth).

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

A browser will open the generated project on http://localhost:9000/ .

![](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150731/20150731152623.png)

<div id="structure"></div>

Directory structure
===================
The generated project has the following structure.


```
.
|-- bower.json                            Bower packages (Client-side libraries)
|-- package.json                          npm packages (Server-side libraries)
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
            |-- thing.controller.js       Server-side controller (API implementation)
            |-- thing.model.js            Server-side DB model
            |-- thing.socket.js           Server-side WebSocket implementation
            `-- thing.integration.js      Server-side test code
```

Client-side codes are deployed under a client directory, and server-side codes are deployed under a server directory. By packaging files into a directory for each feature as a component, components become independent of each other and the whole project is clean and easy to understand.

On client directory, each "app/MODULE" directory stores files for each URL routing as a component. The main files for directories are HTML files (ex: "main.html") and client-side controllers (ex: "main.controller.js"). Other files are client-side routing configurations, test codes, CSS files, etc. Common features for the project are stored under the "components" directory as a subdirectory.

On server directory, each "server/api/MODULE" directory stores files for each URL routing as a component. The main files for the directories are Database models and server-side controllers. Other files are server-side routing configurations, server-side WebSocket implementations, and test codes.

The client and the server send or receive data or events by communicating client-side controllers and server-side controllers using JSON-based HTTP APIs. From MVC model's perspective, the server sees the client as views, and the client sees the server as models.

![](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150709/20150709030234.png)

<div id="generate_server_questions"></div>

Creating server-side question component (model, API, etc.)
======================================

In this QA service, each question is stored as a document in a database.

Generate server-side question-related directory and files (DB mode, server-side controller, etc.) using the generator.

When prompted for a endpoint, set the default ("/api/questions").
The generator generates "server/api/question" directory with files such as "question.controller.js", and "question.model.js". The "/api/question" API is prepared for use.

```shell
% yo angular-fullstack:endpoint question
```


Next, we edit the database model to store question titles, question contents, and the list of answers. MongoDB can directly store arrays or associated arrays as a part of one JSON object. MongoDB itself is flexible schema and does not require pre-defined schemas. But mongoose driver used in Angular Full-Stack generator provides schema as an additional feature to limit or validate fields. So, we define question-related information as a schema.


server/api/question/question.model.js

```javascript
var QuestionSchema = new mongoose.Schema({
  title: String,
  content: String,
});
```

Also, edit test code to adjust edited model. Just replace all the "name" and "info" with "title" and "content", respectively (5 places).

server/api/question/question.integration.js

```javascript
// name: ...
title: ...
// info: ...
content: ...
...
// newQuestion.name....
newQuestion.title....
// question.info....
newQuestion.content....
...
// question.name....
question.title....
// question.info....
question.content....
...
// name: ...
title: ...
// info: ...
content: ...
...
// updatedQuestion.name....
updatedQuestion.title....
// question.info....
updatedQuestion.content....
```


<div id="generate_client_questions"></div>

Creating client-side question-listing, question-creating, and question-showing components (controllers, HTMLs, etc.)
=====================================

Now, we create files for question-listing, question-creating, and question-showing components (controllers, HTMLs).

![](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150731/20150731152639.png)
![](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150814/20150814100206.png)
![](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150814/20150814100204.png)


#### Removing needless files
Remove needless files from the project.

```shell
% rm -r client/app/main
```

#### Generating components

Generate question-related directories and files. Now, we create three directories (questionsIndex, questionsCreate, and questionsShow) for question listing, question-creating, and question showing.


When generator prompted for URL routing as "What will the url of your route be?", type the following URLs.

* questionsIndex (Question listing): /
* questionsCreate (Question creating): /questions/create
* questionsShow (Question showing): /questions/show/:id

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

Directories and files are generated. Now, we implement client-side controllers and HTMLs by editing the files.

#### Editting question-listing routing
To make question-listing page main page, set state for the question-listing page to "main".

client/app/questionsIndex/questionsIndex.js:

```
      .state('main', {
```


#### Editing question-listing controller
On the question-listing controller, we retrieve question listing using "GET /api/questions" API.
Store the retrieved question to $scope variable so that we can refer from the HTML file. Because we use "$http" service, add "$http" to the controller function parameters. The service based on the parameter variable name is assigned to the parameter.

client/app/questionsIndex/questionsIndex.controller.js

```javascript
  .controller('QuestionsIndexCtrl', function ($scope, $http) {
    $http.get('/api/questions').success(function(questions) {
      $scope.questions = questions;
    });
  });
```

#### Editing question-listing HTML file
On the question-listing HTML file, we can refer to "$scope.question" as "questions". By writing an attribute as 'ng-repeat="question in questions"', we can repeatedly output the elements for each question.
Also, we can refer to "$scope" variable such as "{&#x7b;question.title&#x7d;}".

client/app/questionIndex/questionIndex.html

```html
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
```

#### Editing question crating controller
Implement "$scope.submit()" function called when questions are submitted.
The function stores the submitted question ($scope.question) to the server using "POST /api/questions" API.
After the submission, move to submission-listing page using "$location.path('/questions')".


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

#### Editing question-creating HTML file
On the question-creating HTML file, add "ng-submit" attribute to call "submit()" on submissions. Add 'ng-model="question.title"' attribute to input tag to synchronize between input tag and "question.title" variable bi-directionally.

client/app/questionsCreate/questionsCreate.html

```html
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

#### Editing question-showing controller
On the question-showing controller, retrieve question contents and show it.
Question ID can be retrieved from URL("/question/show/:id") by referring to ":id" part as "$stateParams.id".

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

#### Editing question-showing HTML file
On the question-showing HTML file, output question title and contents.
We refer to $scope.question" variable set on controller as "question".


client/app/questionsShow/questionsShow.html

```html
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

client/app/questionsShow/questionsShow.scss

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
However, because we changed the directory structure dramatically, we restart the service to make sure that our changes to code are applied.

```
% grunt serve
```




<div id="add_answers"></div>

Creating answer field
==============================
Now, though we are building the QA service, we can only ask questions and no one can answer them.
So, let's enable to create and display answers.

![](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150814/20150814100156.png)

#### Editing server-side DB model

Edit QuestionSchema to store answers. MongoDB can store JSON object including arrays into a document (corresponding to a record in RDB).

server/api/question/question.model.js

```javascript
var QuestionSchema = new mongoose.Schema({
  title: String,
  content: String,
  answers: [{
    content: String,
  }],
});
```


#### Editing server-side routing

Add answer submission API to the URL routing.

server/api/question/index.js


```javascript
router.post('/:id/answers', controller.createAnswer);
```

#### Editing server-side controller

Implement answer submission API. We can add a value to an array by using MongoDB's '$push' operator.

server/api/question/question.controller.js

```javascript
function createAnswer(req, res) {
  Question.update({_id: req.params.id}, {$push: {answers: req.body}}, function(err, num) {
    if(err) { return handleError(res)(err); }
    if(num === 0) { return res.send(404).end(); }
    exports.show(req, res);
  });
};
```


#### Editing client-side question-showing controller

Add "$scope.submitAnswer()" function called on answer submission. The function sends the answer to the server using "POST /api/questions/QUESTION-ID/answers" API.
Reload the whole question after the submission.

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

#### Editing client-side question-showing HTML
Output the list of answers stored in the question.
Add "ng-submit" attribute to call "$scope.submitAnswer()" on answer submission.

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

Using Markdown
==================
For now, we can use only plain text for questions or answers.
Let's support Markdown like used in Stack Overflow.

We can just add a module and edit tags to support Markdown.

![](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150814/20150814100147.png)
![](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150814/20150814100142.png)

#### Installing angular-pagedown module

Install angular-pagedown module to support Markdown. When prompted for AngularJS version, choose an option to use the latest version.

```shell
% bower install angular-pagedown --save
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

bower.json

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

#### Adding angular-pagedown module to application

To make use of the module, add "ui.pagedown" to application depending modules.

client/app/app.js

```javascript
angular.module('paizaqaApp', [
  ... ,
  'ui.pagedown',
])
```

#### Using pagedown tag

Enable Markdown input. Change from the "textarea" tag to the "pagedown-editor" tag, and set the bound variable by using "content" attribute. For markdown output, change from "{&#x7b;&#x7d;}" to "pagedown-viewer" tag.

client/app/questionsCreate/questionsCreate.html

```html
<!-- <textarea ... ng-model="question.content">...</textarea> -->
<pagedown-editor ng-model="question.content"></pagedown-editor>
```

client/app/questionsShow/questionsShow.html

```html
<!-- {{question.content}} -->
<pagedown-viewer content="question.content"></pagedown-viewer>
...
<!-- {{answer.content}} -->
<pagedown-viewer content="answer.content"></pagedown-viewer>
...
<!-- <textarea ... ng-model="newAnswer.content"></textarea> -->
<pagedown-editor ng-model="newAnswer.content"></pagedown-editor>
```

<div id="tags"></div>

Adding question tags
==================
To make it easy to understand kinds of questions, let's add tags related to questions (ex: "Android", "Objective-C") to each question.

![](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150731/20150731152715.png)
![](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150814/20150814100155.png)

#### Editing database model
Edit QuestionSchema to store tags as an array.

server/api/question/question.model.js

```javascript
var QuestionSchema = new mongoose.Schema({
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

#### Installing ngTagsInput module
Install ngTagsInput module to make it easy to edit or show tags.

```shell
% bower install ng-tags-input --save
```

#### Adding ngTagsInput module to the application modules
To make use of ngTagsInput module, add the module to application depending modules.

client/app/app.js

```javascript
angular.module('paizaqaApp', [
  ...
  'ngTagsInput',
])
```

#### Editing question-creating HTML file
Add question tags input field using "tags-input" tag.
You can also add auto completion by adding "auto-complete" element inside "tags-input" element, but for simplicity, we'll omit the auto completion this time.

client/questionsCreate/questionsCreate.html

```html
    <pagedown-editor ng-model="question.content"></pagedown-editor>
    <h2>Tags:</h2>
    <tags-input ng-model="question.tags">
      <!-- <auto-complete source="loadTags($query)"></auto-complete> -->
    </tags-input>
```

#### Editing question-listing HTML file
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

#### Editing question-showing HTML file
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
For now, we don't know who submits questions and answers. So, let's add user authentication feature.
Only submitted users can edit or remove the articles. Also, store the submission date.

![](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150731/20150731152718.png)

#### Editing server-side database model

Store submitted user's object ID to questions and answers. Specify object's referring model as "ref: 'User'" so that "populate()" function can expand user IDs to User objects.

Though we can manually call pupulate() from each query, in this project, to exapand User object for all the query, hook "find()" and "findOne()" call using "pre()" and call pupulate() in the handler.
"populate('user')" can expands all the fields. But in this project, write "pupulate('user', 'name')" to exapand only 'name' field of the user object.


server/api/question/question.model.js

```javascript
var QuestionSchema = new mongoose.Schema({
  title: String,
  content: String,
  answers: [{
    content: String,
    user: {
      type: mongoose.Schema.ObjectId,
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
    type: mongoose.Schema.ObjectId,
    ref: 'User'
  },
  createdAt: {
    type: Date,
    default: Date.now
  },
});
QuestionSchema.pre('find', function(next){
  this.populate('user', 'name');
  this.populate('answers.user', 'name');
  next();
});
QuestionSchema.pre('findOne', function(next){
  this.populate('user', 'name');
  this.populate('answers.user', 'name');
  next();
});
```


#### Editing server-side API routing

On the server-side API routing, add "auth.isAuthenticated()" as an Express middleware to the URL resource requiring authentication so that the server-side controller can refer to the current login user as "req.user". 
Also, add an answer-deleting API(DELETE /:id/answers/:answerId).

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

#### Editing server-side controller
Change the query to return the last 20 questions.
"sort({createdAt: -1})" sort by created time in descending order and "limit(20)" returns the first 20 objects.
After creating the query, call "execAsync()" to execute the query.

server/api/question/question.controller.js

```javascript
export function index(req, res) {
  Question.find().sort({createdAt: -1}).limit(20).execAsync()
    ...
```

Change the question-creating API to save a user as a part of question.

server/api/question/question.controller.js

```javascript
export function create(req, res) {
  req.body.user = req.user;
  Question.create(req.body, ...
...
export function createAnswer(req, res) {
  req.body.user = req.user;
  Question.update(...
```

On question-updating and question-destroying API, verify that current login user ID is the same as the question's user ID so that
only submitted users can edit or delete the articles.

server/api/question/question.controller.js

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
// Updates an existing Question in the DB
export function update(req, res) {
  ...
    .then(handleEntityNotFound(res))
    .then(handleUnauthorized(req, res))
    ...
...
// Deletes a Question from the DB
export function destroy(req, res) {
  ...
    .then(handleEntityNotFound(res))
    .then(handleUnauthorized(req, res))
    ...
```

Implement answer-destroying API. Delete the answer specified by answer's ID and answer's user ID using MongoDB '$pull' operator.

```javascript
export function destroyAnswer(req, res) {
  Question.update({_id: req.params.id}, {$pull: {answers: {_id: req.params.answerId , 'user': req.user._id}}}, function(err, num) {
    if(err) { return handleError(res)(err); }
    if(num === 0) { return res.send(404).end(); }
    exports.show(req, res);
  });
};
```

Implement answer-updating API. On the MongoDB query, we can refer the matching index of the array (answer array in this case) as '$'.
Add a condition to match current login user and answer's user ID so that only submitted user can update the answer.

```javascript
export function updateAnswer(req, res) {
  Question.update({_id: req.params.id, 'answers._id': req.params.answerId}, {'answers.$.content': req.body.content, 'answers.$.user': req.user.id}, function(err, num){
    if(err) { return handleError(res)(err); }
    if(num === 0) { return res.send(404).end(); }
    exports.show(req, res);
  });
};
```

#### Editing client-side question-listing HTML

Add user name and created time to the question.

client/app/questionsIndex/questionsIndex.html

```html
          <a ng-href="/questions/show/{{question._id}}" style="font-size: large">{{question.title}}</a>
          <div class="clearfix"></div>
          <div style="float: right;">
            by {{question.user.name}}
             · {{question.createdAt}}
          </div>
```

#### Editing client-side question-showing controller
Implement the question- and answer-deletion functions ("deleteQuestion()", "deleteAnswer()") and updating functions ("updateQuestion()", "updateAnswer()"). 
The deleting functions call "DELETE /api/questions/:id" and "DELETE /api/questions/:id" API.
The updating functions call "PUT /api/questions/:id" and "PUT /api/questions/:id/answers/:id" API.

Also, implement "isOwner()" function which checks whether the user of question or answer matches the current login user. We can use "Auth.isLoggedIn()" to check whether the user is logged in, and "Auth.getCurrentUser()._id" to get current login user ID.
To user "$location" and "Auth" modules, add "$location" and "Auth" parameter to the controller function. AngularJS automatically assigns the services to the parameters from the parameter names.

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

#### Editing client-side question-showing HTML file

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

Also, add editing button. "editing" variable is true while editing. Use "ng-show"/"ng-if" to switch element by "editing" variable. Although we are using the same "editing" variable for the question and answers, answers are under an element with the "ng-repeat" attribute and "ng-repeat" create separate scope for each item. So, each "editing" actually refers to different variables for the question and each answer.

client/app/questionsShow/questionsShow.html

```html
      <h1>
        <div ng-if="! editing">{{question.title}}</div>
        <input type=text ng-model="question.title" ng-if=" editing">
      </h1>
      ...
  <pagedown-viewer content="question.content" ng-if="!editing"></pagedown-viewer>
  <pagedown-editor ng-model="question.content" ng-if=" editing"></pagedown-editor>
  <button type="submit" class="btn btn-primary" ng-click="editing=false;updateQuestion()" ng-show=" editing">Save</button>
  <a ng-click="editing=!editing;" ng-show="isOwner(question) && !editing">Edit</a>
  ...
    <div class="answer">
      ...
      <pagedown-viewer content="answer.content" ng-if="!editing"></pagedown-viewer>
      <pagedown-editor ng-model="answer.content" ng-if=" editing"></pagedown-editor>
      <button type="submit" class="btn btn-primary" ng-click="editing=false;updateAnswer(answer)" ng-show=" editing">Save</button>
      <a ng-click="editing=!editing;" ng-show="isOwner(answer) && !editing">Edit</a>
    ...
```

#### Editing client-side question-creating controller

If the current user is not authenticated, move to the login page.

client/app/questionsCreate/questionsCreate.controller.js

```javascript
  .controller('QuestionsCreateCtrl', function ($scope, $http, $location, Auth) {
    if(! Auth.isLoggedIn()){
      $location.path('/login');
      return;
    }
    ...
```

#### Editing server-side test
In this project, remove routing test.

```shell
% rm server/api/question/index.spec.js
```

For APIs requiring authentication, before each test, login and set authentication information before the test.

server/api/question/question.integration.js

```javascipt
var User = require('../user/user.model');
...
describe('Question API:', function() {
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
  describe('POST /api/questions', function() {
    ...    
        .post('/api/questions')
        .set('authorization', 'Bearer ' + token)
    ...
  describe('PUT /api/questions/:id', function() {
    ...
        .put('/api/questions/' + newQuestion._id)
        .set('authorization', 'Bearer ' + token)
    ...
  describe('DELETE /api/questions/:id', function() {
    ...
        .delete('/api/questions/' + newQuestion._id)
        .set('authorization', 'Bearer ' + token)
    ...
        .delete('/api/questions/' + newQuestion._id)
        .set('authorization', 'Bearer ' + token)
    ...
  /* describe('PUT /api/things/:id', function() {
  }); */
```

<div id="validation"></div>

Input validation
==============
For now, we can submit a form with an empty question, or an empty answer. Let's validate inputs not to submit without filling the field.

![valication](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150731/20150731160812.gif)

#### Installing ngMessage module
Install ngMessage module for the validation as a client-side library.

```shell
% bower install angular-messages --save
```


#### Adding ngMessage module to application module
Add the ngMessage module to the application depending modules to use the module.

client/app/app.js

```javascript
angular.module('paizaqaApp', [
  ...
  'ngMessages',
])
```

#### Editing client-side HTML files
Add validations (ex: "required") to the input fields. Also, to refer the validation results, add name (with "name" attribute) to the form and add name(with "name" attribute) and model (with "ng-model" attribute) to each input field. The validation result is stored as "FORM-NAME.FIELD-NAME.$error", and output the result using ng-messages/ng-message attributes in which only matching elements are shown. The whole form validation result is stored as "FORM-NAME.$invalid", and we can disable the submit button when invalid.

client/app/questionsCreate/questionsCreate.html

```html
  <form name="form" ng-submit="submit()">
    <h2>Title:</h2>
    <input type="text" class="form-control" ng-model="question.title" name="question_title" required>
    <span class="text-danger" ng-messages="form.question_title.$error">
      <span ng-message="required">Required</span>
    </span>
    <span class="text-success" ng-show="form.question_title.$valid">OK</span>
    <br>
    <h2>Question:</h2>
    <pagedown-editor ng-model="question.content" ng-model="question.content" name="question_content" required></pagedown-editor>
    <span class="text-danger" ng-messages="form.question_content.$error">
      <span ng-message="required">Required</span>
    </span>
    <span class="text-success" ng-show="form.question_content.$valid">OK</span>
    <h2>Tags:</h2>
    <tags-input ng-model="question.tags">
      <!-- <auto-complete source="loadTags($query)"></auto-complete> -->
    </tags-input>
    <input type="submit" class="btn btn-primary" ng-disabled="form.$invalid" value="Post question">
  </form>
```

client/app/questionsCreate/questionsShow.html

```html
    <pagedown-editor ng-model="newAnswer.content" name="answerEditor" required></pagedown-editor>
    <input type="submit" class="btn btn-primary" ng-disabled="answerForm.$invalid" value="Submit your answer">
```


<div id="fromnow_filter"></div>

Time formatting filter
============
For now, the time format for created time is UTC. Let's change it to show the time from now like in Stack Overflow.

![](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150731/20150731152721.png)

#### Installing Moment.js library
Install Moment.js library for time formatting as a client-side library.

```shell
% bower install --save momentjs
```

Add a "moment-with-locales.min.js" file for I18N (no need for English).

client/index.html

```html
    <!-- build:js({client,node_modules}) app/vendor.js -->
      <!-- bower:js -->
      ...
      <!-- endbower -->
      <script src="bower_components/moment/min/moment-with-locales.min.js"></script>
      <script src="socket.io-client/socket.io.js"></script>
    <!-- endbuild -->
```

#### Generating filter

Generate a filter boilerplate.

```shell
% yo angular-fullstack:filter fromNow
% grunt injector
```

#### Implementing filter

Format time using the Moment.js's fromNow() function. You can optionally use locale() function to set language.

client/app/fromNow/fromNow.filter.js

```javascript
    return function (input) {
      return moment(input).locale(window.navigator.language).fromNow();
    };
```

#### Using filter

Change time format from UTC to time from now using the fromNow filter we created. To use filter, change from "{&#x7b;EXPRESSION&#x7d;}" to "{&#x7b;EXPRESSION|FILTER&#x7d;}". In this case, we change from "{&#x7b;EXPRESSION&#x7d;}"を"{&#x7b;EXPRESSION|fromNow&#x7d;}".

client/app/questionsIndex/questionsIndex.html

```html
<!-- Old: {{question.createdAt}} -->
{{question.createdAt|fromNow}}

```

client/app/questionsCreate/questionsShow.html

```html
<!-- Old: {{question.createdAt}} -->
{{question.createdAt|fromNow}}
```

```html
<!-- Old: {{answer.createdAt}} -->
{{answer.createdAt|fromNow}}

```

#### Changing test code

Fix the failing test.

Change test code to test that the fromNow filter with the current time(Date.now()) returns 'a few seconds ago'.

client/app/fromNow/fromNow.filter.spec.js

```javascript
  it('return "a few seconds ago" for now', function () {
    expect(fromNow(Date.now())).toBe('a few seconds ago');
  });
```

<div id="add_comments"></div>

Adding comments
================
Now, let's add comment fields for questions and answers.

![](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150814/20150814100154.png)

#### Editing server-side database model
On QuestionSchema, store comments as an array inside the each question and answer. Each comment holds the created time and submitted user.


server/api/question/question.model.js

```javascript
var QuestionSchema = new mongoose.Schema({
  ...
  answers: [{
    ...
    comments: [{
      content: String,
      user: {
        type: mongoose.Schema.ObjectId,
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
      type: mongoose.Schema.ObjectId,
      ref: 'User'
    },
    createdAt: {
      type: Date,
      default: Date.now,
    }
  }],
});

QuestionSchema.pre('find', function(next){
  this.populate('user', 'name');
  this.populate('comments.user', 'name');
  this.populate('answers.user', 'name');
  this.populate('answers.comments.user', 'name');
  next();
});
QuestionSchema.pre('findOne', function(next){
  this.populate('user', 'name');
  this.populate('comments.user', 'name');
  this.populate('answers.user', 'name');
  this.populate('answers.comments.user', 'name');
  next();
});
```

#### Editing server-side routing
Add the following APIs to create, update, and delete a comment on a question or an answer.

| Method | URL | Description |
|--------|-----|-----------|
| POST   | /:id/comments | Create a comment on a question |
| PUT    | /:id/comments/:commentId | Update a comment on a question |
| DELETE | /:id/comments/:commentId | Delete a comment on a question |
| POST   | /:id/answers/:answerId/comments | Create a comment on an answer |
| PUT    | /:id/answers/:answerId/comments/:commentId | Update a comment on an answer |
| DELETE | /:id/answers/:answerId/comments/:commentId | Delete a comment on an answer |

server/api/question/index.js

```javascript
router.post('/:id/comments', auth.isAuthenticated(), controller.createComment);
router.put('/:id/comments/:commentId', auth.isAuthenticated(), controller.updateComment);
router.delete('/:id/comments/:commentId', auth.isAuthenticated(), controller.destroyComment);

router.post('/:id/answers/:answerId/comments', auth.isAuthenticated(), controller.createAnswerComment);
router.put('/:id/answers/:answerId/comments/:commentId', auth.isAuthenticated(), controller.updateAnswerComment);
router.delete('/:id/answers/:answerId/comments/:commentId', auth.isAuthenticated(), controller.destroyAnswerComment);
```


#### Editing server-side controller

On the question-listing API, expand a user ID of a comment to a user object using populate().

server/api/question/question.controller.js

```javascript
exports.show = function(req, res) {
  Question.findById(req.params.id).populate('user', 'name').populate('comments.user', 'name').populate('answers.user', 'name').populate('answers.comments.user', 'name').exec(function (err, question) {
    ...
```

Implement APIs to create, update, or delete a comment. To create a comment, add a comment to the comment array of the question document using '$push' operator. To delete a comment, delete a comment from the comment array of the question document. To update a comment, specify the updating index of the comment array of the question document using '$'. To update a comment on an answer, because we update an item inside an array of an array and only one '$' can be used to specify the index, we need to iterate each item for the array.


server/api/question/question.controller.js

```javascript
/* comments APIs */
export function createComment(req, res) {
  req.body.user = req.user.id;
  Question.update({_id: req.params.id}, {$push: {comments: req.body}}, function(err, num){
    if(err) {return handleError(res)(err); }
    if(num === 0) { return res.send(404).end(); }
    exports.show(req, res);
  })
}
export function destroyComment(req, res) {
  Question.update({_id: req.params.id}, {$pull: {comments: {_id: req.params.commentId , 'user': req.user._id}}}, function(err, num) {
    if(err) { return handleError(res)(err); }
    if(num === 0) { return res.send(404).end(); }
    exports.show(req, res);
  });
}
export function updateComment(req, res) {
  Question.update({_id: req.params.id, 'comments._id': req.params.commentId}, {'comments.$.content': req.body.content, 'comments.$.user': req.user.id}, function(err, num){
    if(err) { return handleError(res)(err); }
    if(num === 0) { return res.send(404).end(); }
    exports.show(req, res);
  });
}

/* answersComments APIs */
export function createAnswerComment(req, res) {
  req.body.user = req.user.id;
  Question.update({_id: req.params.id, 'answers._id': req.params.answerId}, {$push: {'answers.$.comments': req.body}}, function(err, num){
    if(err) {return handleError(res)(err); }
    if(num === 0) { return res.send(404).end(); }
    exports.show(req, res);
  })
}
export function destroyAnswerComment(req, res) {
  Question.update({_id: req.params.id, 'answers._id': req.params.answerId}, {$pull: {'answers.$.comments': {_id: req.params.commentId , 'user': req.user._id}}}, function(err, num) {
    if(err) { return handleError(res)(err); }
    if(num === 0) { return res.send(404).end(); }
    exports.show(req, res);
  });
}
export function updateAnswerComment(req, res) {
  Question.find({_id: req.params.id}).exec(function(err, questions){
    if(err) { return handleError(res)(err); }
    if(questions.length === 0) { return res.send(404).end(); }
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
          if(err) { return handleError(res)(err); }
          if(num === 0) { return res.send(404).end(); }
          exports.show(req, res);
          return;
        });
      }
    }
    if(!found){
      return res.send(404).end();
    }
  });
}
```


#### Editing client-side controller

Add functions to add, update, or delete a comment that send the request to the server.

client/app/questionsShow/questionsShow.controller.js

```javascript
    $scope.newComment = {};
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

#### Editing client-side question-showing HTML file

Output comments' content, created time, and user for the question or the answers in the question object. Also, add forms to submit new comments.


client/app/questionsShow/questionsShow.html

```html
  <div class="text-right">by <a ng-href="/users/{{question.user._id}}">{{question.user.name}}</a> · {{question.createdAt|fromNow}}</div>
  &nbsp;
  <div class="comment">
    <div ng-repeat="comment in question.comments">
      <hr/>
      <button ng-if="isOwner(comment)" type="button" class="close" ng-click="deleteComment(comment)">&times;</button>

      <pagedown-viewer content="comment.content" ng-if="!editing"></pagedown-viewer>
      <pagedown-editor ng-model="comment.content" ng-if=" editing"></pagedown-editor>
      <button type="submit" class="btn btn-primary" ng-click="editing=false;updateComment(comment)" ng-show=" editing">Save</button>
      <a ng-click="editing=!editing;" ng-show="isOwner(comment) && !editing">Edit</a>

      <div class="text-right" style="vertical-align: bottom;">by <a ng-href="/users/{{comment.user._id}}">{{comment.user.name}}</a> · {{comment.createdAt|fromNow}}</div>
      <div class="clearfix"></div>
    </div>
    <hr/>
    <a ng-click="editNewComment=!editNewComment;">add a comment</a>
    <form ng-if="editNewComment" name="commentForm">
      <pagedown-editor ng-model="newComment.content" editor-class="'comment-wmd-input'"
        ng-model="newComment.content" name="commentEditor" required>
      </pagedown-editor>
      <button type="button" class="btn btn-primary" ng-click="submitComment()" ng-disabled="commentForm.$invalid">Add Comment</button>
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
 
        <pagedown-viewer content="comment.content" ng-if="!editing"></pagedown-viewer>
        <pagedown-editor ng-model="comment.content" ng-if=" editing"></pagedown-editor>
        <button type="submit" class="btn btn-primary" ng-click="editing=false;updateAnswerComment(answer, comment)" ng-show=" editing">Save</button>
        <a ng-click="editing=!editing;" ng-show="isOwner(comment) && !editing">Edit</a>

        <div class="text-right">by <a ng-href="/users/{{question.user._id}}">{{comment.user.name}}</a> · {{comment.createdAt|fromNow}}</div>
        <div class="clearfix"></div>
      </div>
      <hr/>
      <a ng-click="editNewAnswerComment=!editNewAnswerComment;answer.newAnswerComment={}">add a comment</a>
      <form ng-if="editNewAnswerComment" name="answer_{{answer.id}}_comment">
        <hr/>
        <pagedown-editor ng-model="answer.newAnswerComment.content" editor-class="'comment-wmd-input'"
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

client/app/questionsShow/questionsShow.scss

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
Adding a feature to star or unstar questions, answers, and comments on questions or answers.

![](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150731/20150731152723.png)


#### Editing server-side database model

Store the list of starring users for a question, an answer, and a comment on a question or answer to a "stars" field of a question document as an array.

server/api/question/question.model.js

```javascript
var QuestionSchema = new mongoose.Schema({
  ...
  answers: [{
    ...
    comments: [{
      ...
      stars: [{
        type: mongoose.Schema.ObjectId,
        ref: 'User'
      }],
      ...
    }],
    stars: [{
      type: mongoose.Schema.ObjectId,
      ref: 'User'
    }],
    ...
  }],
  ...
  comments: [{
    ...
    stars: [{
      type: mongoose.Schema.ObjectId,
      ref: 'User'
    }],
    ...
  }],
  stars: [{
    type: mongoose.Schema.ObjectId,
    ref: 'User'
  }],
  ...
});
```

#### Adding server-side routing

Add the following APIs to star or unstar a question, an answer, or a comment on a question or an answer.

| Method | URL | Description |
|--------|-----|-------------|
| POST   | /:id/star | Star a question |
| DELETE | /:id/star | Unstar a question |
| POST   | /:id/answers/:answerId/star | Star an answer |
| DELETE | /:id/answers/:answerId/unstar | Unstar an answer |
| POST   | /:id/comments/:commentId/star | Star a comment on a question |
| DELETE | /:id/comments/:commentId/unstar | Unstar a comment on a question |
| POST   | /:id/answers/:answerId/comments/:commentId/star | Star a comment on an answer |
| DELETE | /:id/answers/:answerId/comments/:commentId/star | Unstar a comment on an answer |

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



#### Editing server-side database model

On the database model, for questions, answers, and comments on questions or answers, store the list of starred users as an array.
On "update()" function, we can refer the matched index of an array using "$". For the list of a starred users of a comment array of an answer array, because we can only use one "$", we need to iterate the array explicitly.

server/api/question/question.controller.js

```javascript
/* star/unstar question */
export function star(req, res) {
  Question.update({_id: req.params.id}, {$push: {stars: req.user.id}}, function(err, num){
    if(err) { return handleError(res)(err); }
    if(num === 0) { return res.send(404).end(); }
    exports.show(req, res);
  });
}
export function unstar(req, res) {
  Question.update({_id: req.params.id}, {$pull: {stars: req.user.id}}, function(err, num){
    if(err) { return handleError(res, err); }
    if(num === 0) { return res.send(404).end(); }
    exports.show(req, res);
  });
}

/* star/unstar answer */
export function starAnswer(req, res) {
  Question.update({_id: req.params.id, 'answers._id': req.params.answerId}, {$push: {'answers.$.stars': req.user.id}}, function(err, num){
    if(err) { return handleError(res)(err); }
    if(num === 0) { return res.send(404).end(); }
    exports.show(req, res);
  });
}
export function unstarAnswer(req, res) {
  Question.update({_id: req.params.id, 'answers._id': req.params.answerId}, {$pull: {'answers.$.stars': req.user.id}}, function(err, num){
    if(err) { return handleError(res)(err); }
    if(num === 0) { return res.send(404).end(); }
    exports.show(req, res);
  });
}

/* star/unstar question comment */
export function starComment(req, res) {
  Question.update({_id: req.params.id, 'comments._id': req.params.commentId}, {$push: {'comments.$.stars': req.user.id}}, function(err, num){
    if(err) { return handleError(res)(err); }
    if(num === 0) { return res.send(404).end(); }
    exports.show(req, res);
  });
}
export function unstarComment(req, res) {
  Question.update({_id: req.params.id, 'comments._id': req.params.commentId}, {$pull: {'comments.$.stars': req.user.id}}, function(err, num){
    if(err) { return handleError(res)(err); }
    if(num === 0) { return res.send(404).end(); }
    exports.show(req, res);
  });
}

/* star/unstar question answer comment */
var pushOrPullStarAnswerComment = function(op, req, res) {
  Question.find({_id: req.params.id}).exec(function(err, questions){
    if(err) { return handleError(res)(err); }
    if(questions.length === 0) { return res.send(404).end(); }
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
          if(err) { return handleError(res)(err); }
          if(num === 0) { return res.send(404).end(); }
          exports.show(req, res);
          return;
        });
      }
    }
    if(!found){
      return res.send(404).end();
    }
  });
};
export function starAnswerComment(req, res) {
  pushOrPullStarAnswerComment('$push', req, res);
}
export function unstarAnswerComment(req, res) {
  pushOrPullStarAnswerComment('$pull', req, res);
}
```

#### Editing client-side question-showing controller

Add functions to star, unstar, or check staring status for a question, an answer, or a comment on a question or an answer.
We use a sub-pathname parameter to use the common function for questions, answers, or comments.

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


#### Editing client-side question-showing HTML file

Show the staring status as the star icon. We can click the star icon to star or unstar.

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
        <div ng-if="! editing">{{question.title}}</div>
```

client/app/questionsShow/questionsShow.html

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

      <pagedown-viewer content="comment.content" ng-if="!editing"></pagedown-viewer>
```

client/app/questionsShow/questionsShow.html

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

        <pagedown-viewer content="comment.content" ng-if="!editing"></pagedown-viewer>
```

#### Editing client-side question-listing controller

Add "isStar()" function to show the staring status on question listing.

client/app/questionsIndex/questionsIndex.controller.js

```javascript
  .controller('QuestionsIndexCtrl', function ($scope, $http, Auth, $location) {
    ...
    $scope.isStar = function(obj){
      return Auth.isLoggedIn() && obj && obj.stars && obj.stars.indexOf(Auth.getCurrentUser()._id)!==-1;
    };
```

#### Editing client-side question-listing HTML file

On question listing, show the number of staring users for the question and the number of answers.

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


#### Editing client-side routing

Assign the following URLs for all questions, my questions, and starred questions.

| URL | Description |
|-----|-------------|
| / | All questions |
| /users/:userId | My questions |
| /users/:userId/starred | Starred questions |

Use the same controller and HTML template and change the searching query by "query" variable.
For my questions listing, the query checks whether the user of the question is the same as the current login user.
For starred questions listing, the query checks whether the starred user for the question, the answers, or the comments contains current login user.

client/app/questionsIndex/questionsIndex.js


```javascript
...
  .config(function ($stateProvider) {
    $stateProvider
      .state('main', {
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

#### Editing client-side question-listing controller
Send the query set on routing to the server.

client/app/questionsIndex/questionsIndex.controller.js

```javascript
  .controller('QuestionsIndexCtrl', function ($scope, $http, Auth, query) {
    $http.get('/api/questions', {params: {query: query}}).success(function(questions) {
```

#### Editing client-side question-listing controller test
Add empty a "query" to test code not to cause an error.

client/app/questionsIndex/questionsIndex.controller.spec.js

```javascript
    QuestionsIndexCtrl = $controller('QuestionsIndexCtrl', {
      $scope: scope,
      query: {},
    });
```

#### Editing server-side controller
Send the received query to the database.

server/api/question/question.controller.js

```javascript
export function index(req, res) {
  var query = req.query.query && JSON.parse(req.query.query);
  Question.find(query).sort(...
```

#### Editing client-side Navbar controller
On Navbar, add links for all questions, my questions, and ffffffstarred questions.
Because we need to change the URL or enable/disable for links before login or logout, use functions instead of variables for that information on menu items.

client/components/navbar/navbar.controller.js

```javascript
  constructor(Auth) {
    this.menu = [
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
    this.isLoggedIn = Auth.isLoggedIn;
    this.isAdmin = Auth.isAdmin;
    this.getCurrentUser = Auth.getCurrentUser;
  }
  ...
```

#### Editing client-side Navbar HTML
Retrieve linked URLs using "$scope.item.link()" function.
Also, show the link only when "$scope.item.show()" returns true.


client/components/navbar/navbar.html

```html
      <ul class="nav navbar-nav">
        <li ng-repeat="item in nav.menu" ng-class="{active: isActive(item.link())}" ng-show="item.show()">
            <a ng-href="{{item.link()}}">{{item.title}}</a>
        </li>
      </ul>
```

<div id="search"></div>

Search
============
With MongoDB's full-text search, let's add a feature to search on question titles, contents, comments, or answers.

![](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150731/20150731152725.png)

#### Editing client-side Navbar HTML file

Add serach box to Navbar. Call "search()" function on search.

client/components/navbar/navbar.html:

```html
      <form class="navbar-form navbar-left" role="search" ng-submit="nav.search(keyword)">
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

#### Editing client-side Navbar controller

client/components/navbar/navbar.controller.js

```javascript
  constructor(Auth, $state) {
    this.search = function(keyword) {
      $state.go('main', {keyword: keyword}, {reload: true});
    };
    ...
```

#### Editing server-side database model

To add index for a full-text search for question titles, question contents, comments, and answers, use "QuestionSchema.index()" function and specify search target fields as 'text'.
Although MongoDB can automatically set index name, because MongoDB does not work as intended if the length of the name is long and exceeds 128 bytes, let's explicitly specify the index name (ex: 'question_schema_index').
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


#### Editing client-side question-listing routing

To accept search keyword as "keyword" URL parameter, add "/?keyword" to the "url" field of the routing information.

client/app/questionsIndex/questionsIndex.js

```javascript
      .state('main', {
        url: '/?keyword',
        ...
```

#### Editing client-side question-showing controller
Set the query using MongoDB's '$text' and '$search' parameters to search by 'keyword' parameter.


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


Japanese search
==============
MongoDB's full-text search only supports Latin languages, and does not support other languages such as Japanese.
Let's support Japanese search by using Japanese tokenizer "TinySegmenter".

![](http://cdn-ak.f.st-hatena.com/images/fotolife/p/paiza/20150731/20150731160824.gif)

#### Installing TinySegmenter library
Install TinySegmenter as a server-side library.


```shell
% npm install --save r7kamura/tiny-segmenter
```

On the server-side DB model, add "searchText" field to store tokenized text.

server/api/question/question.model.js

```javascript
var QuestionSchema = new mongoose.Schema({
  ...
  searchText: String,
});

```

Add "searchText" field to the text index.

server/api/question/question.model.js

```javascript
QuestionSchema.index({
  ...
  'searchText': 'text',
}, {name: 'question_schema_index'});
```

Add a function to tokenize, and call it before saving by using "pre('save')" hook.
Also, add a static function to the question model by adding the function to "QuestionSchema.statics".

server/api/question/question.model.js

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

Because "pre('save')" hook is not called by "update()" call, explicitly call "updateSearchText()" function to tokenize and update the full-text index.

server/api/question/question.controller.js

```javascript
export function createAnswer(req, res) {
    ...
    exports.show(req, res);
    Question.updateSearchText(req.params.id);
  });
};
export function destroyAnswer(req, res) {
  ...
    exports.show(req, res);
    Question.updateSearchText(req.params.id);
  });
};
export function updateAnswer(req, res) {
    ...
    exports.show(req, res);
    Question.updateSearchText(req.params.id);
  });
};
...
/* comments APIs */
export function createComment(req, res) {
    ...
    exports.show(req, res);
    Question.updateSearchText(req.params.id);
  })
};
export function destroyComment(req, res) {
    ...
    exports.show(req, res);
    Question.updateSearchText(req.params.id);
  });
};
export function updateComment(req, res) {
    ...
    exports.show(req, res);
    Question.updateSearchText(req.params.id);
  });
};
...
/* answersComments APIs */
export function createAnswerComment(req, res) {
    ...
    exports.show(req, res);
    Question.updateSearchText(req.params.id);
  })
};
export function destroyAnswerComment(req, res) {
    ...
    exports.show(req, res);
    Question.updateSearchText(req.params.id);
  });
};
export function updateAnswerComment(req, res) {
  ...
          exports.show(req, res);
          Question.updateSearchText(req.params.id);
  ...
};
```

To re-create index, drop the question collection and restart the server(grunt serve).
```
% mongo
> use paizaqa-dev
> db.questions.drop()
% grunt serve
```


<div id="infinite_scroll"></div>

Infinite scroll
============
For now, we can list only the last 20 questions. Let's add an infinite scroll to show older questions.


#### Installing ngInfiniteScroll library

Install ngInfiniteScroll module for infinite scroll as a client-side library.

```shell
% bower install --save ngInfiniteScroll
```


#### Adding ngInfiniteScroll to the application modules
Add ngInfiniteScroll to the application depending modules to use.

client/app/app.js

```javascript
angular.module('paizaqaApp', [
  ...
  'infinite-scroll',
 ])
```

#### Editing client-side question-listing controller

To list older questions on a scroll, implement "nextPage()" function. To retrieve questions older than the current oldest question ID, specify the query as "{_id: {$lt: lastID}}". 
Store the loading status as "$scope.busy", and the status of whether more data is available as "$scope.noMoreData".

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

#### Editing client-side question listing
To load older questions on scroll to the bottom, add "infinite-scroll" attribute to call "nextPage()" function.
Disable scroll when loading or when there is no more data.
Show "Loading data" on loading by adding an element with "ng-show='busy'", so the element is shown only when the "busy" variable is "true". 

client/app/questionsIndex/questionsIndex.html

```html
<div class="container" infinite-scroll='nextPage()' infinite-scroll-disabled='busy || noMoreData'>
  ...
  <div ng-show='busy'>Loading data...</div>
</div>
```

<div id="oauth"></div>

SNS authentication
=========
When using SNS authentication(Facebook, Twitter, Google), set API key and SECRET key. See
[an instruction in the first article](http://engineering.paiza.io/entry/2015/07/08/153011#sns_link) for details.

For Facebook authentication, because API specification changed on Graph API 2.4 (on July 9th, 2015), we need to explicitly specify fields to use on "profileFields" as follows.

<!--
server/auth/facebook/passport.js

x```
exports.setup = function (User, config) {
  passport.use(new FacebookStrategy({
      profileFields: ['displayName', 'name', 'profileUrl', 'id', 'email',  'photos', 'gender', 'locale', 'timezone', 'updated_time', 'verified'],
      clientID: ...
    ...
x```
-->

<div id="deploy"></div>

Deploying
============
Now, let's deploy the application to Heroku.
To configure Heroku deployment, use "yo angular-fullstack:heroku" command.
Also, install a MongoDB module MongoLab to the Heroku application, as MongoLab has a free plan.

```shell
% yo angular-fullstack:heroku
% cd dist
% heroku addons:add mongolab
```

From the next deployment, use "grunt" to build the application, and use "grunt buildcontrol:heroku" to deploy the application.

```shell
% grunt
% grunt buildcontrol:heroku
```

The application is deployed. Open http://APPLICATION.herokuapp.com/ to see the application !

When it does not work, see Heroku logs.

```shell
% cd dist
% heroku logs
```

For browsing MongoDB, GUI tools such as MongoHub are convenient.
We can retrieve the URL for the MongoDB database from Heroku using "heroku config" command.

```shell
% heroku config
...
MONGOLAB_URI: mongodb://USERNAME:PASSWORD@HOSTNAME:PORT/DATABASE
...

```

<div id="summary"></div>

Summary
===============
I introduced how to build a QA service using a MEAN stack, AngularJS Full-Stack generator.
Using MEAN stack, we can quickly build a sophisticated web service, from client-side logic to server-side logic, by only using JavaScript.
Using the generator, we can get best practice boilerplate codes, and just edit the codes to create services. It is especially helpful for startups or prototypes where it is necessary to create and change services quickly by trial and error.

Lets' come up with ideas, and build your own services!

I welcome any feedback such as errors, suggestion, or anything you noticed about this articles as comments!

I'll continue writing articles to build web services using MEAN stack.

<br/>

<table width="100%" border="0" cellspacing="0" cellpadding="0">
<thead>
  <tr>
    <td colspan="2" style="background-color: darkblue; color: white;"><div style="font-size: small; font-weight: bold;">MEAN stack development articles</div></td>
  </tr>
</thead>
<tbody>
  <tr>
    <td></td>
    <td><a href="http://engineering.paiza.io/entry/2015/07/08/153011">Building full-stack web service - MEAN stack development(1)</a></td>
  </tr>
  <tr>
    <td></td>
    <td><a href="http://engineering.paiza.io/entry/2015/07/09/154028">Building Twitter-like full-stack web service in 1 hour - MEAN stack development (2)</a></td>
  </tr>
  <tr>
    <td>*</td>
    <td><ax href="http://engineering.paiza.io/entry/2016/03/10/115345">Building a QA web service in an hour - MEAN stack development(3)</ax></td>
  </tr>

</tbody>
</table>


