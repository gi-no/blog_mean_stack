<!-- Title: Building Twitter-like full-stack web service in 1 hour - MEAN stack development (2) -->
<!-- URL: http://engineering.paiza.io/entry/2015/07/09/154028 -->



<iframe src="https://player.vimeo.com/video/133009564?autoplay=1&title=0&byline=0&portrait=0" width="500" height="408" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>


[f:id:paiza:20140712194904j:plain] (by Yoshioka Tsuneo, [twitter:@yoshiokatsuneo] at https://paiza.IO/)

(Japanese article is available <a href="http://paiza.hatenablog.com/entry/2015/07/09/1時間でTwitter風フルスタック・Webサービスを作る！-_MEANス">here</a>.)

[In the previous article](http://engineering.paiza.io/entry/2015/07/08/153011), I introduced a MEAN stack, Yeoman-based AngularJS Full-Stack generator, and the way how to install, run, edit, debug and deploy programs. MEAN stack is a web development package of MongoDB, Express, AngularJS, NodeJS. You can easily build interactive and intuitive full-stack web service by just using one language: JavaScript.

In this article, we'll build a more practical real web service!

The web service is Twitter-like service where users can post and list messages.
We can build full-fledged, nearly production ready web service based by editing some JavaScript or HTML code generated. Let's try!

[f:id:paiza:20150706130351p:plain]

[Demo: http://paizatter.herokuapp.com](http://paizatter.herokuapp.com)

The web service has features like below:

* Sign Up, and Login
* Post, remove, or list messages
* Search posted messages
* Infinite scrolling list
* Starred messages (Add, remove, or list)

You can download the source code below. But, I suggest writing code by your hands to get more feeling.

[https://github.com/gi-no/paizatter](https://github.com/gi-no/paizatter)


Contents
====================
* [Install MEAN stack](#install)
* [Create a new project](#new_project)
* [List messages](#list_message)
* [Change order of the list](#change_order)
* [User authentication](#user_authentication)
* [Edit CSS](#edit_css)
* [Deploy](#deploy)
* [SNS authentication](#sns_authentication)
* [Debug](#debug)
* [Create time format filter](#time_filter)
* [Starred messages](#starred)
* [List user messages, starred messages](#user_messages)
* [Search](#search)
* [Infinite scroll](#infinite_scroll)
* [Re-deploy](#re_deploy)

<div id="install"></div>
Install MEAN stack
=======================
Install Yeoman-based AngularJS Full-Stack generator(generator-angular-fullstack) following instructions in a previous article.


<div id="new_project"></div>
Create a new project
=======================

At first, let's create a new project. Create a project directory and run "yo"(Yeoman) command. I named the project as "paizatter".

```shell
$ mkdir paizatter
$ cd paizatter
$ yo angular-fullstack paizatter
```

There are multiple configurations to choose.
Let's just choose default settings except SNS setting where we enable all the SNS.

```
- Would you like to include additional oAuth strategies? 
 ◉ Google
 ◉ Facebook
❯◉ Twitter
```

Just after a minute, many project files are generated. Following is some of project files related to this article.

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

Client-side codes are under "client" directory, and server-side codes are under "server" directory.

On "client/app" directory, each page have own directory(ex: "client/app/main"). The directory pack JavaScript code(controller), HTML file(view), URL routing configuration file, CSS files, test file to make it easy to maintain.

On "server/api" directory, each subdirectory has own JavaScript API code(controller), Web socket code, URL routing configuration, test code, and DB model.

[f:id:paiza:20150709030234p:plain]

Client-side controllers communicate with server-side controllers using server API, and update HTML template or handle events. Server-side controllers communicates with client-side controller using server API, and retrieve or update data from/to MongoDB through DB model.

If we think about MVC, from clients' point of view, servers are like models, from servers' point of view, clients are like views.



Default npm packages on Angular Full-stack generator is a bit old, so update to latest packages using "npm-check-updates" .

```shell
% sudo npm install -g npm-check-updates
% npm-check-updates -u
% npm install
```

Now, start the server.

```shell
% grunt serve
```

<div id="list_message"></div>
List messages
==============
Generated project have a controller(client/app/main/main.controller.js) that owns "things" array object. The main page list the "things" object.

In this project, we use the "thing" object to store messages.

Open HTML file and edit a "div" element with "container" class.

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

Now, the main page shows input form and message list.

You see "ng-repeat" or "{{expression}}" that is not used in HTML. Those are AngularJS syntax to embed JavaScript variables on HTML templates.

"ng-repeat" is an AngularJS syntax to expand array object in HTML templates. You can write as "ng-repeat=ITEM in ARRAY" to output each object in the array. You can use "{{expression}}" syntax to embed variable or simple expression(Angular Expression) to HTML templates.

[f:id:paiza:20150706134637p:plain]


<div id="change_order"></div>
Change order of the list
================
Now, we see the message list, but new messages are appended on the bottom of the list, instead of on the top of the list. Let's append new message on the top of the list like Twitter. Also, let's show only last 20 messages instead of all the messages.

On WebSocket code, use "push" instead of "unshift" to append new item on the top of the messages.

client/components/socket/socket.service.js:

```javascript
      syncUpdates: ...
        socket.on(...
          ...
          // array.push(item);
          array.unshift(item);
```

Callback code inside "socket.on()" is called using WebSocket when message list is updated. Thanks to WebSocket, we can automatically update message list without manually reloading page when other users added message, 

To change the order on loading messages when initially loading page or reloading page, edit the server-side controller for listing message.

server/api/thing/thing.controller.js:

```javascript
// Get list of things
exports.index = function(req, res) {
  Thing.find().sort({_id:-1}).limit(20).exec(function (err, things) {
    if(err) { return handleError(res, err); }
    return res.json(200, things);
  });
};
```

Use sort() function of mongoose(MongoDB middleware), to sort on descendant order by creation time.
MongoDB has "_id" field on every document(RDB's record), and "_id" field is ordered by creation time. So, we can just sort by "_id" field to sort by creation time.

limit() function limit the number of object to return. When the query is built, call exec() function to call query. Query result is returned as an argument of the callback function. So, just return the result to the client.

<div id="user_authentication"></div>
User authentication
=================
Now, we can list the message, we don't see who post the messages. Also, only posted user should be able to delete the message.

So, let's add user authentication.

Sign Up and Login feature is already generated from the template, so we just need to add authentication for features, and show user name on the messages.



#### Server-side model schema

Store user ID and message together. MongoDB itself has flexible schema, but AngularJS FullStack generator also uses mongoose as a driver. Mongoose has features like removing needless fields on save, hook functions or expand related document.

On mongoose schema configuration for messages(ThingSchema), add user ID to message schema. Also, add creation time.

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

Now, "name" field store a message, "user" field store User's ObjectID. "ref: 'User'" relate the ObjectID to User collection, and enables to expand using populate() function. "createAt" field have "Date.now" function as a default value to set creation time automatically.

#### Server-side API routing configuration
For API Requests requiring authentication, add "auth.isAuthenticated()" middleware to the routings. By that, posting or deleting messages from unauthorized users are prohibited, and request object(req) have user field(req.user) to store User object.

server/api/thing/index.js:

```javascript
var auth = require('../../auth/auth.service');

router.get('/', controller.index);
router.get('/:id', controller.show);
router.post('/', auth.isAuthenticated(), controller.create);
router.delete('/:id', auth.isAuthenticated(), controller.destroy);
```
On the routing file above, specify function called when each URL is called.
To add authentication, specify auth.isAuthenticated() as middleware. Also, remove needless put/patch routing.


#### Edit server-side controller(create function)
On server-side controller, add user object to user field of creating document.(req.body.user = req.user)
Because "req.user" already contains User object, just set it to Thing.create argument("req.body") to save user.

server/api/thing/thing.controller.js:

```javascript
// Creates a new thing in the DB.
exports.create = function(req, res) {
  req.body.user = req.user;
  Thing.create(req.body, function(err, thing) {
```


#### Edit server-side controller to show messages
Now, API functions to output message(index(), show()) just returns ObjectID of User instead of User object itself. Use populate() function to expand User object.
populate('user') expand all fields of User object. Specify populate('user','name') to just expand needed field('name').

server/api/thing/thing.controller.js:

```javascript
// Get list of things
exports.index = function(req, res) {
  Thing.find().sort({_id:-1}).limit(20).populate('user',' username').exec(function (err, things) {
```

```javascript
// Get a single thing
exports.show = function(req, res) {
  Thing.findById(req.params.id).populate('user','user').exec(function (err, thing) {
```


#### Edit server-side controller to delete messages.
On deletion, validate that the posting user and the current user is the same before deletion.

server/api/thing/thing.controller.js:

```javascript
// Deletes a thing from the DB.
exports.destroy = ...
...
    if(!thing) { ... }
    if(thing.user.toString() !== req.user._id.toString()){
      return res.send(403);
    }
```

####Edit server-side WebSocket code
Call populate() to expand User on update notification through WebSocket.

server/api/thing/thing.socket.js:

```javascript
function onSave(socket, doc, cb) {
  doc.populate('user', 'name', function(){
    socket.emit('thing:save', doc);    
  })
}
```

####Edit client-side controller
Add isMyTweet() function to check whether the message is mine or not.

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

On the above controller function, arguments of controller function specify modules to use. So, add "Auth" to the controller function arguments.

Variables or functions stored in $scope object can be referred on HTML code, add function as "$scope.isMyTweet".
"isMyTweet" function checks whether the message's user ID is the same as the current user or not. 
Also, to call authentication function from HTML templates, add isLoggedIn/getCurrentUser to $scope object.

#### Edit client-side HTML template
On message listing, add username and creation time.

client/app/main/main.html:

```html
  <div ng-repeat="thing in awesomeThings">
    <div class="row">
      {{thing.user.name}} - {{thing.name}} ({{thing.createdAt}})
      <button ng-if="isMyTweet(thing)" type="button" class="close" ng-click="deleteThing(thing)">&times;</button>
    </div>
  </div>
```


#### Test
Now, all authentication feature is implemented, let's test it.

If you post without Login, you are redirected to Sign Up page. Posted message contains the username. You can only delete your message (using cross("x") button).

[f:id:paiza:20150706135436p:plain]

<div id="edit_css"></div>
Edit CSS
============
Current message list has no decoration. Add CSS to decorate message.

#### CSSARROW
Choose arrow CSS from http://cssarrowplease.com . Just choose favorite style and append it to main.scss.

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

Also, add margin or set font.

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

####Edit HTML file
Edit HTML file to apply CSS styles.

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
Deploy
================
At this point, basic features are done!

Let's deploy for now.

```shell
% yo angular-fullstack:heroku
% cd dist
% heroku addons:add mongolab
```

"yo angular-fullstack:heroku" command setup deploy environment for Heroku.
Also, add MongoDB module to Heroku. Heroku provides MongoHQ and MongoLab as MongoDB add-on. Let's add MongoLab add-on because MongoLab has a free plan.

Now, we deployed the web service to Heroku !
For next deployments, use "grunt" command to build distribution package, and "grunt buildcontrol:heroku" for deployment.

```shell
% grunt
% grunt buildcontrol:heroku
```

Now, it's time to open a browser to use the web service!

http://APPLICATION-NAME.herokuapp.com/

<div id="sns_authentication"></div>
SNS authentication
========
To use SNS authentication(Facebook, Twitter, Google), set up API key and SECRET key. Please refer [instruction in the previous article](http://engineering.paiza.io/entry/2015/07/08/153011#sns_link).

<div id="debug"></div>
Debug
=========
In case you failed deployment, check out server log file. Please refer [instruction in the previous article](http://engineering.paiza.io/entry/2015/07/08/153011#debug) for details.

% cd dist
% heroku logs

For about MongoDB operation, GUI tool like MongoHub is helpful. MongoDB URL can be retrieved from Heroku configuration.

% heroku config
...
MONGOLAB_URI:    mongodb://Username:Password@Hostname:Port/Database
...

<div id="time_filter"></div>
Create time format filter
==============

Now, message creation time is shown in UTC. Let's change to show time from now like Twitter.

#### Install momentjs

We use a time formatting JavaScript library "momenjs" as a client-side library. Install the library using bower.

```shell
% bower install --save momentjs
% grunt wiredep
```

"--save" options saves the package name to "bowser.json", and "grunt wiredep" adds script tags to load the library to "index.html".


#### Create fromNow AngularJS filter

Create "fromNow" AngularJS filter. Filter is an AngularJS feature to format the value. So, let's create a filter to format time as time from now.

Generate fromNow filter using a generator. The generator will create a directory and put filter code and test code under the directory.
For now, when we created new directory, we need to run "grunt injector" or "grunt serve" to load JavaScript files. ( [grunt-contrib-watch/issues/166](https://github.com/gruntjs/grunt-contrib-watch/issues/166) )

```shell
% yo angular-fullstack:filter fromNow
% grunt injector
```

Edit the filter code to call momentjs's fromNow() function.

client/app/fromNow/fromNow.filter.js

```javascript
    return function (input) {
      return moment(input).fromNow();
    };
```

Now, the fromNow filter is implemented.

So, let's use the filter on HTML template.
We can use filter just by adding "|filter" at the end of "{{expression}}" style expression. Now, change from "{{thing.createdAt}}" to "{{thing.createdAt|fromNow}}".


client/app/main/main.html:

```html
        <span style="float: right;">({{thing.createdAt|fromNow}})</span>
```

Now, message creation time is formatted as time from now like "~minutes ago".

[f:id:paiza:20150706145049p:plain]

### Edit test code

Now, we need to edit test code because we edited filter code.
Add moment.js to karma.conf.js so that test code load moment.js.

karma.conf.js:

```javascript
    files: [
        ...
      'client/bower_components/momentjs/moment.js',
      ...
    ],
```

Then, edit test code to test that fromNow filter for current time returns 'a few seconds ago'.

client/app/fromNow/fromNow.filter.spec.js

```javascript
  it('return "a few seconds ago" for now', function () {
    expect(fromNow(Date.now())).toBe('a few seconds ago');
  });
```

Now, run tests, and see there is no error.

```shell
% grunt test
```



#### I18N
To format in user language, use "moment-with-locales.min.js". On "client/index.html", add script tag after "<!-- endbower -->".

client/index.html

```html
      <!-- endbower -->
      <script src="bower_components/momentjs/min/moment-with-locales.min.js"></script>
```

Change the fromNow filter to use browser's language (window.navigator.language).

client/app/fromNow/fromNow.filter.js

```javascript
    return function (input) {
      return moment(input).locale(window.navigator.language).fromNow();
    };
```

Now, time is formatted using browser's language (Ex: "〜分前" in Japanese).

[f:id:paiza:20150706145342p:plain]

<div id="starred"></div>
Starred messages
===============

Let's add a feature to star/unstar messages.

#### On servser-side DB model, add starred user to message schema

Store starred users to messages. On MongoDB, you can store array as a part of a document. So, we'll store starred users as a part of a message. On message schema, add "stars" field with array type to store the list of user ObjectIDs.

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

#### Add server-side URL routing

To star/unstar message, add two API(star/unstar) to server-side URL routing. To allow star/unstar only for authenticated user, add "isAuthenticated" to routing middleware.

server/api/thing/index.js:

```
router.put('/:id/star', auth.isAuthenticated(), controller.star);
router.delete('/:id/star', auth.isAuthenticated(), controller.unstar);
```

#### Implement server-side API.

Implement star/unstar API. We can use update() function with "{$push/$pull: {field name: value}}" to insert or remove item to/from array inside a document. Implementation for two APIs are same except "$push" or  "$pull". Call show() function at the end of API implementation to return a updated message.

server/api/thing/thing.controller.js:

```javascript
exports.star = function(req, res) {
  Thing.update({_id: req.params.id}, {$push: {stars: req.user}}, function(err, num){
    if (err) { return handleError(res, err); }
    if(num===0) { return res.send(404); }
    exports.show(req, res);
  });
};
```

```javascript
exports.unstar = function(req, res) {
  Thing.update({_id: req.params.id}, {$pull: {stars: req.user}}, function(err, num){
    if (err) { return handleError(res, err); }
    if(num === 0) { return res.send(404); }
    exports.show(req, res);
  });
};
```

#### Implement client-side controller

Make star/unstar function on the client-side controller that just calls server side star/unstar API. Those two functions are same except for requesting method("put" or "delete").
Also, add "isMyStart()" function to see whether current user starred message or not.

client/app/main/main.controller.js:


```javascript
    $scope.starThing = function(thing) {
      $http.put('/api/things/' + thing._id + '/star').success(function(newthing){
        $scope.awesomeThings[$scope.awesomeThings.indexOf(thing)] = newthing;
      });
    };
```

```javascript
    $scope.unstarThing = function(thing) {
      $http.delete('/api/things/' + thing._id + '/star').success(function(newthing){
        $scope.awesomeThings[$scope.awesomeThings.indexOf(thing)] = newthing;
      });
    };
```

```javascript
    $scope.isMyStar = function(thing){
      return Auth.isLoggedIn() && thing.stars && thing.stars.indexOf(Auth.getCurrentUser()._id)!==-1;
    }
```

#### Edit HTML template file

Edit HTML file to add star icon, and call starThing() to star on click. If the message is already starred, call unstarThing() to unstar.


client/app/main/main.html:

```html
<div class="container">
  ...
  <div class="row">
    <div ng-repeat="thing in awesomeThings" class="tweet">
      ...
      <div class="arrow_box col-xs-10">
        <button ng-if="isMyTweet(thing)" type="button" class="close" ng-click="deleteThing(thing)">&times;</button>
        <button ng-if=" isMyStar(thing)" type="button" class="close" ng-click="unstarThing(thing)">
          <span class="glyphicon glyphicon-star" style="color: #CF7C00;" ></span>
        </button>
        <button ng-if="!isMyStar(thing)" type="button" class="close" ng-click="starThing(thing)"  >
          <span class="glyphicon glyphicon-star-empty"></span>
        </button>
```

Now, we can star/unstar messages.

<div id="user_messages"></div>
List user messages, starred messages
=================================

[f:id:paiza:20150706145600p:plain]

Until now, message list shows all users' messages. Let's add a feature to list only user messages or only starred messages.

#### Add client-side routing
Create new URLs to show each user's messages or starred messages.

* User messages: /users/ユーザID
* Starred messages: /users/ユーザID/starred

Client-side routing is set using $stateProvider.state function. Add above URLs to routing with the same controller(MainCtrl) and template(main.html). To filter messages, we set query. Add "query" to "resolve" field. Filter by user for user messages, and filter by stars field for starred messages.

On MongoDB, we can write queries using JavaScript. So, we can just transfer the query to MongoDB through Server-side API to filter messages.

Note that if you put "/users/:userId" first on the routing, "starred" will be part of userId. So, put "/users/:userId/stared" before that.

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
            return {stars: $stateParams.userId}
          }
        }
      })
      .state('user', {
        url: '/users/:userId',
        templateUrl: 'app/main/main.html',
        controller: 'MainCtrl',
        resolve: {
          query: function($stateParams){
            return {user: $stateParams.userId}
          }
        }
      })
      ;
  });
```

#### Edit client-side controller

Add a query parameter to server-API requests. Add "query" to controller function, and add the query to $http.get() argument.

client/app/main/main.controller.js:

```javascript
  .controller('MainCtrl', function ($scope, $http, socket, Auth, query) {
  ...
    $http.get('/api/things', {params: {query: query}}).success(function(awesomeThings) {
```

#### Edit server-side controller

On the server-side controller, just transfer the received query to MongoDB by passing query as find() arguments.

server/api/thing/thing.controller.js

```javascript
exports.index = ...
  var query = req.query.query && JSON.parse(req.query.query);
  Thing.find(query).sort...
```


#### Add nav links to navbar

Until now, navbar has only one link "Home". Change it to three links like "All", "Mine", "Starred".

Add link items to $scope.menu array. Enable "Mine" or "Starred" link only for logged in users. To switch link dynamically before or after login, set "link" field(for URL) and "show" field functions.

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
        'link': function(){return '/users/' + Auth.getCurrentUser()._id + '/starred'},
        'show': Auth.isLoggedIn,
      },
    ];
```

On navbar HTML file, change from "link" variable to "link()" function. Set item.show() on "ng-show"  to display on when show() returns true.

client/components/navbar/navbar.controller.html:

```html
        <li ng-repeat="item in menu" ng-class="{active: isActive(item.link())}" ng-show="item.show()">
            <a ng-href="{{item.link()}}">{{item.title}}</a>
        </li>

```

#### Edit client-side HTML template
Show user message on click user name. Just add link to user message URL(/users/userID).

client/app/main/main.html:

```html
        <a ng-href="/users/{{thing.user._id}}">{{thing.user.name}}</a>
```

#### Edit test code
Edit test code to add dummy query parameter.

client/app/main/main.controller.spec.js:

```javascript
    MainCtrl = $controller('MainCtrl', {
      $scope: scope,
      query: null,
    });
```

Now, we can see my or other users messages, or starred messages.


<div id="search"></div>
Search
============

[f:id:paiza:20150706145744p:plain]

Let's add a feature to search messages. MongoDB has a full text search feature, so let's use it.

#### Change client-side URL routing configuration

Use URLs below with "keyword" for searching.

* All: /?keyword=KEYWORD
* User: /users/:userId?keyword=KEYWORD
* Starred: /users/:userId/starred?keyword=KEYWORD

On routing configuration, write url field like "XXX?keyword" so that we can use "keyword" as a parameter.

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
      

#### Add search box

Add search box to Navbar. Set "search(keyword)" to ng-submit attribute so that submitting keyword invoke the search function.


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


#### Navbar controller: change routing state on search.

On navbar controller, add a search function to change the URL state to have searching keyword specified.

client/components/navbar/navbar.controller.js:


```javascript
    $scope.search = function(keyword) {
      $state.go('main', {keyword: keyword});        
    };
```

This works. But, it always searches from all messages. It would be nice if we can also search from user messages or starred messages. Change the search function to keep URL state(main, user, or starred) on search. Don't forget to add '$state' to NavbarCtrl function argument to use the variable.

client/components/navbar/navbar.controller.js:


```javascript
  .controller('NavbarCtrl', function ($scope, $location, Auth, $state) {
    $scope.search = function(keyword) {
      if ($state.current.controller == 'MainCtrl'){
        $state.go($state.current.name, {keyword: keyword}, {reload: true});        
      }else{
        $state.go('main', {keyword: keyword}, {reload: true});        
      }
    };
```

### Regular expression search

Now, let's try to search using a regular expression. Use MongoDB's  '$regex' operator to search by the regular expression.

```javascript
    var keyword = $location.search().keyword;
    if(keyword){
      query = _.merge(query, {name: {$regex: keyword, $options: 'i'}});
    }
    $http.get('/api/things', {params: {query: query}})...
```

### Full-text search

Regular expression search works, but it will be getting slow if we have many messages.

So, let's use full-text search MongoDB provides. MongoDB have '$text' / '$search' operator for full text search. For full-text search, we don't(can't) specify field to search.



```javascript
    var keyword = $location.search().keyword;
    if(keyword){
      query = _.merge(query, {$text: {$search: keyword}});
    }
    $http.get('/api/things', {params: {query: query}})...
```

#### Edit server-side model

For the full-text search, add 'text' index to the searching field on the schema.

server/api/thing/thing.mode.js:

```javascript
ThingSchema.index({name: 'text'});
```

Now, we can search by word like "Development". (We cannot search by substring match.)


####  Japanese search
MongoDB's full-text searching only support Latin languages, and does not support other languages like Japanese.

We can use ElasticSearch or other engines. But for now, we use TinySegmenter to tokenize Japanese.

Add "tokenizedName" field to store and indexing tokenized message.

```javascript
var ThingSchema = new Schema({
  name: String,
  tokenizedName: String,
  ...
}
...
ThingSchema.index({tokenizedName: 'text', name: 'text'});
```

Drop "things" collection for re-indexing.
```shell
% mongo
> use APPLICATION-dev
> db.things.drop()
```


Install TinySegmenter using npm as a server-side library.
```shell
% npm install --save r7kamura/tiny-segmenter
```

Tokenize message on save, and join with space, and save to "tokenizedName" field. You can hook on save by calling "pre('save', callback)" on the schema.

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

Now, we can search by Japanese words, we can search message "吾輩は猫である" by a word "我輩".

<div id="infinite_scroll"></div>
Infinite scroll
============

Now, we can only see last 20 messages, and there is no way to see older messages.

Let's add infinite scroll to see older messages like Twitter.

#### On client-side, install "ngInfiniteScroll" library
Install an AngularJS library "ngInfiniteScroll" for infinite scroll.

```javascript
% bower install --save ngInfiniteScroll
% grunt wiredep
```

#### Load ngInfiniteScroll
To use ngInfiniteScroll module, add it to AnguarJS application module dependency.

client/app/app.js:

```javascript
angular.module('paizatterApp', [
   ... ,
   'infinite-scroll'
]);
```

#### Edit HTML file

To use ngInfiniteScroll module, on div tag of "container" class, add "infinite-scroll" attribute to call a function("nextPage()") on scroll. Set flags("busy", "noMoreData") to "infinite-scroll-disabled" attribute not to scroll while loading or if no more message available.

At the end of HTML, output "Loading data" while loading.

client/app/main/main.html:

```html
<div class="container" infinite-scroll='nextPage()' infinite-scroll-disabled='busy || noMoreData'>
  ...
  <div ng-show='busy'>Loading data...</div>
</div>
```

#### Edit client-side controller

On $scope variable, create "busy" field to store for the loading state, and "noMoreData" flag to store whether all the message is loaded or not.

On scroll, we need to load message older than last message. Add "{_id: {$lt: lastId}}" to query. 

On initial loading, if there are messages fewer than 20, set "noMoreData" flag.

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
      $http.get('/api/things', {params: {query: query}}).success(function(awesomeThings_) {
        $scope.awesomeThings = $scope.awesomeThings.concat(awesomeThings_);
        $scope.busy = false;
        if(awesomeThings_.length == 0){
          $scope.noMoreData = true;
        }
      });
    };

```

Now, we can load messages older than 20 last messages, using infinite scroll.

#### Edit karma.conf
Add "ngInfiniteScroll" library to karma.conf to load on test.

karma.conf.js:

```javascript
    files: [
      ...
      'client/bower_components/ngInfiniteScroll/build/ng-infinite-scroll.js',
      ...
    ]
```

Check test result.

```(shell)
% grunt test
```

<div id="re_deploy"></div>
Re-deploy
===============

Finally, we built all the features!
Let's re-deploy the latest version to Heroku.

```shell
% grunt
% grunt buildcontrol:heroku
```

Open browser and see it works!

http://APPLICATION.herokuapp.com/

<div id="summary"></div>
Summary
=================
In this article, we created Twitter-like full-stack web service using a MEAN stack, Angular Full-Stack generator. 
We have not edited many lines of codes. But, we could build almost full-fledged, real web service.

By MEAN stack, we can easily create Web services just using JavaScript.
Let's come up with ideas and build your own web services!

I welcome your feedbacks about the instruction.

I'll continue writing articles about creating web service using MEAN stack.
