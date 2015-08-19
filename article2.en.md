<!-- Title: Building Twitter-like full-stack web service in 1 hour - MEAN stack development (2) -->
<!-- URL: http://engineering.paiza.io/entry/2015/07/09/154028 -->



<iframe src="https://player.vimeo.com/video/133009564?autoplay=1&title=0&byline=0&portrait=0" width="500" height="408" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>


[f:id:paiza:20140712194904j:plain]  (by Yoshioka Tsuneo, [twitter:@yoshiokatsuneo] at https://paiza.IO/)

(Japanese article is available <a href="http://paiza.hatenablog.com/entry/2015/07/09/1時間でTwitter風フルスタック・Webサービスを作る！-_MEANス">here</a>.)

[In the previous article](http://engineering.paiza.io/entry/2015/07/08/153011), I introduced a MEAN stack, Yeoman-based AngularJS Full-Stack generator, and explained how to install, run, edit, debug, and deploy programs. MEAN stack is a web development package of MongoDB, Express, AngularJS, and Node.js. You can easily build interactive and intuitive full-stack web services by just using one language: JavaScript.

In this article, we'll build a more practical real web service!

The web service is a Twitter-like service where users can post and list messages.
We can build a full-fledged, nearly production-ready web service based by editing some JavaScript or HTML code generated. Let's try!

[f:id:paiza:20150706130351p:plain]

[Demo: http://paizatter.herokuapp.com](http://paizatter.herokuapp.com)

The web service has features like below:

* Sign Up, and Login
* Post, remove, or list messages
* Search posted messages
* Infinite scrolling list
* Starred messages (Add, remove, or list)

You can download the source code below. But, I suggest writing code by your hands to have a better understanding for the codes.

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
Install Yeoman-based AngularJS Full-Stack generator(generator-angular-fullstack) following [the instructions in the previous article](http://engineering.paiza.io/entry/2015/07/08/153011#install).

Confirm that installed AngularJS Full-Stack generator is ver3.0.0 or later.

```shell
$ npm ls -g generator-angular-fullstack
/usr/local/lib
└── generator-angular-fullstack@3.0.0-rc4 
```

If it is older than ver3.0.0, update to the latest version.

```shell
$ sudo npm update -g generator-angular-fullstack
```

<div id="new_project"></div>
Create a new project
=======================

First, let's create a new project. Create a project directory and run "yo"(Yeoman) command. I named the project "paizatter".

```shell
$ mkdir paizatter
$ cd paizatter
$ yo angular-fullstack paizatter
```

There are multiple configurations from which to choose.
Let's just choose default settings except for the SNS setting where we enable all the SNS.

```
- Would you like to include additional oAuth strategies? 
 ◉ Google
 ◉ Facebook
❯◉ Twitter
```

After a minute, many project files are generated. The following are some of project files related to this article.

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
            `-- thing.integration.js      Server-side test code
```

Client-side codes are under the "client" directory, and server-side codes are under the "server" directory.

In the "client/app" directory, each page has its own directory (ex: "client/app/main"). The directory pack a JavaScript code (controller), a HTML file (view), a URL routing configuration file, a CSS files, and a test file to make it easy to maintain.

In the "server/api" directory, each subdirectory has own JavaScript API code(controller), Web socket code, URL routing configuration, test code, and DB model.

[f:id:paiza:20150709030234p:plain]

Client-side controllers communicate with server-side controllers using the server APIs, and they update HTML page or handle events. Server-side controllers communicate with client-side controllers using server APIs, and they retrieve or update data from/to MongoDB through the DB model.

If we think about MVC, from the clients' point of view, servers are like models. From servers' point of view, clients are like views.



The default npm packages on Angular Full-stack generator are a bit old, so update to the latest packages using "npm-check-updates" .

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

Open the HTML file and edit a "div" element with "container" class.

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

Now, the main page shows the input form and message list.

You see "ng-repeat" or "{{expression}}" that is not used in HTML. Those are AngularJS syntax to embed JavaScript variables on the HTML templates.

"ng-repeat" is an AngularJS syntax to expand an array object in HTML templates. You can write "ng-repeat=ITEM in ARRAY" to output each object in the array. You can use "{{expression}}" syntax to embed a variable or simple expression(Angular Expression) in the HTML templates.

[f:id:paiza:20150706134637p:plain]


<div id="change_order"></div>
Change order of the list
================
Now, we see the message list, but new messages are appended on the bottom of the list, instead of on the top of the list. Let's append a new message on the top of the list like Twitter does. Also, let's show only the last 20 messages instead of all the messages.

On WebSocket code, use "push" instead of "unshift" to append a new item on the top of the messages.

client/components/socket/socket.service.js:

```javascript
      syncUpdates: ...
        socket.on(...
          ...
          // array.push(item);
          array.unshift(item);
```

Callback code inside "socket.on()" is called using WebSocket when the message list is updated. Thanks to WebSocket, we can automatically update the message list without manually reloading page when other users add a message, 

To change the order on loading messages when initially loading the page or reloading the page, edit the server-side controller for the listing message.

server/api/thing/thing.controller.js:

```javascript
// Gets a list of Things
exports.index = function(req, res) {
  Thing.find().sort({_id:-1}).limit(20).execAsync()
    .then(responseWithResult(res))
    .catch(handleError(res));
};
```

Use sort() function of mongoose (MongoDB middleware), to sort in descending order by creation time.
MongoDB has "_id" field on every document (RDB's record), and the "_id" field is ordered by creation time. So, we can just sort by "_id" field to sort by creation time.

limit() function limits the number of object to return. When the query is built, call exec() function to call query. The query result is returned as an argument of the callback function. So, just return the result to the client.

<div id="user_authentication"></div>
User authentication
=================
Now, we can list the messages, but we don't see who posted the messages. Also, only posted user should be able to delete his/her message.

So, let's add user authentication.

Sign Up and Login feature is already generated from the template, so we just need to add authentication for features and show the user name on the messages.



#### Server-side model schema

Store the user ID and the message together. MongoDB itself has flexible schema, but AngularJS Full-Stack generator also uses mongoose as a driver. Mongoose has features such as removing needless fields on save, hook functions, and expanding related documents.

On the mongoose schema configuration for messages(ThingSchema), add user ID to message schema. 
The "name" field stores a message, "user" field stores User's ObjectID. "ref: 'User'" relates the ObjectID to the User collection, and enables to expand using populate() function.

Also, add creation time.
The "createAt" field have "Date.now()" function as a default value to set creation time automatically.

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


And, find() or findOne() query just returns the ObjectID of User instead of the User object itself. Use the populate() function to expand User object.
populate('user') expand all fields of the User object. Specify populate('user','name') to just expand a needed field('name').

Although we can expand on each query, to expand for all query, use "pre()" to hook all 'find()', 'findOne()' call and call populate().

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


#### Server-side API routing configuration
For APIs requiring authentication, add "auth.isAuthenticated()" middleware to the routings. In this way, posting or deleting messages from unauthorized users is prohibited, and request object ("req") have user field ("req.user") to store User object.

server/api/thing/index.js:

```javascript
var auth = require('../../auth/auth.service');

router.get('/', controller.index);
router.get('/:id', controller.show);
router.post('/', auth.isAuthenticated(), controller.create);
router.delete('/:id', auth.isAuthenticated(), controller.destroy);
```
On the routing file above, we can specify a function called when each URL is requested.
To add authentication, specify auth.isAuthenticated() as a middleware. Also, remove needless put/patch routing.


#### Edit server-side controller(create function)
On the server-side controller, add the user object to "user" field of creating document (req.body.user = req.user).
Because "req.user" already contains the user object, just set it to Thing.create argument ("req.body") to save user.

server/api/thing/thing.controller.js:

```javascript
// Creates a new Thing in the DB
exports.create = function(req, res) {
  req.body.user = req.user;
  Thing.createAsync(req.body)
  ...
```


#### Edit server-side controller to delete messages.
On deletion, validate that the posting user and the current user are the same before deletion.

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


####Edit client-side controller
Add isMyTweet() function to check whether the message is of current user or not.

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

On the above controller function, arguments of the controller function specify modules to use. So, add "Auth" to the controller function arguments.

Variables or functions stored in $scope object can be referred on HTML code, so add a function as "$scope.isMyTweet".
"isMyTweet" function checks whether the message's user ID is the same as the current user ID or not. 
Also, to call authentication function from HTML templates, add isLoggedIn/getCurrentUser to $scope object.

#### Edit client-side HTML template
On the message listing, add the username and creation time.

client/app/main/main.html:

```html
  <div ng-repeat="thing in awesomeThings">
    <div class="row">
      {{thing.user.name}} - {{thing.name}} ({{thing.createdAt}})
      <button ng-if="isMyTweet(thing)" type="button" class="close" ng-click="deleteThing(thing)">&times;</button>
    </div>
  </div>
```

#### Edit server-side test
For APIs requiring authentication, before each test, login and set authentication information before the test. Also, remove a test for "PUT API" which we don't use.

server/api/thing/thing.integration.js:

```javascipt
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
  /* describe('PUT /api/things/:id', function() {
  }); */
```

#### Test
Now, all authentication feature have been implemented, so let's test it.

If you post without Login, you are redirected to Sign Up page. The posted message contains the username. You can only delete your message (using the cross("x") button).

[f:id:paiza:20150706135436p:plain]

<div id="edit_css"></div>
Edit CSS
============
The current message list has no decoration. Add CSS to decorate message.

#### CSSARROW
Choose an arrow CSS from http://cssarrowplease.com . Just choose your favorite style and append it to "main.scss".

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

####Edit the HTML file
Edit the HTML file to apply CSS styles.

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

The "yo angular-fullstack:heroku" command will set up a deploy environment for Heroku.
Also, add MongoDB module to Heroku. Heroku provides MongoHQ and MongoLab as MongoDB add-ons. Let's add MongoLab add-on because MongoLab has a free plan.

Now, we deployed the web service to Heroku !
For next deployments, use the "grunt" command to build a distribution package, and "grunt buildcontrol:heroku" for deployment.

```shell
% grunt
% grunt buildcontrol:heroku
```

Now, it's time to open a browser to use the web service!

http://APPLICATION-NAME.herokuapp.com/

<div id="sns_authentication"></div>
SNS authentication
========
To use SNS authentication(Facebook, Twitter, Google), set up API key and SECRET key. Please refer to [the instructions in the previous article](http://engineering.paiza.io/entry/2015/07/08/153011#sns_link).

<div id="debug"></div>
Debug
=========
In case you failed deployment, check out the server log file. Please refer to [the instruction in the previous article](http://engineering.paiza.io/entry/2015/07/08/153011#debug) for details.

```shell
% cd dist
% heroku logs
```

For about MongoDB operation, GUI tools like MongoHub are helpful. MongoDB URL can be retrieved from the Heroku configuration.

% heroku config
...
MONGOLAB_URI:    mongodb://Username:Password@Hostname:Port/Database
...

<div id="time_filter"></div>
Create time format filter
==============

Now, the message creation times are shown in UTC. Let's change it to show time from now like Twitter does.

#### Install momentjs

We use a time formatting JavaScript library "momenjs" as a client-side library. Install the library using bower.

```shell
% bower install --save momentjs
% grunt wiredep
```

"--save" options saves the package name to "bowser.json", and "grunt wiredep" adds script tags to load the library to "index.html".


#### Create fromNow AngularJS filter

Create "fromNow" AngularJS filter. Filter is an AngularJS feature to format the value. So, let's create a filter to format time as time from now.

Generate fromNow filter using a generator. The generator will create a directory and put the filter code and test code under the directory.
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

Now, the fromNow filter has been implemented.

So, let's use the filter on the HTML template.
We can use the filter just by adding "|filter" at the end of "{{expression}}" style expression. Now, change from "{{thing.createdAt}}" to "{{thing.createdAt|fromNow}}".


client/app/main/main.html:

```html
        <span style="float: right;">({{thing.createdAt|fromNow}})</span>
```

Now, the message creation times are formatted as time from now like "~minutes ago".

[f:id:paiza:20150706145049p:plain]

### Edit test code

Now, we need to edit the test code because we edited filter code.


Edit the test code to test so that fromNow filter for the current time returns 'a few seconds ago'.

client/app/fromNow/fromNow.filter.spec.js

```javascript
  it('return "a few seconds ago" for now', function () {
    expect(fromNow(Date.now())).toBe('a few seconds ago');
  });
```

Now, run tests, and confirm there is no error.

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

Change the fromNow filter to use the the browser's language (window.navigator.language).

client/app/fromNow/fromNow.filter.js

```javascript
    return function (input) {
      return moment(input).locale(window.navigator.language).fromNow();
    };
```

Now, time is formatted using the browser's language (Ex: "〜分前" in Japanese).

[f:id:paiza:20150706145342p:plain]

<div id="starred"></div>
Starred messages
===============

Let's add a feature to star/unstar messages.

#### On servser-side DB model, add starred user to message schema

Store starred users to messages. On MongoDB, you can store an array as a part of a document. So, we'll store starred users as a part of a message. On the message schema, add "stars" field with array type to store the list of user ObjectIDs.

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

To star/unstar a message, add two APIs(star/unstar) to the server-side URL routing. To allow star/unstar only for authenticated users, add "isAuthenticated" to the routing middleware.

server/api/thing/index.js:

```
router.put('/:id/star', auth.isAuthenticated(), controller.star);
router.delete('/:id/star', auth.isAuthenticated(), controller.unstar);
```

#### Implement server-side API.

Implement star/unstar API. We can use the update() function with "{$push/$pull: {field name: value}}" to insert or remove an item to/from an array inside a document. Implementation for the two APIs are same except for "$push" and  "$pull". Call "show()" function at the end of API implementation to return an updated message.

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

#### Implement client-side controller

Make star/unstar functions on the client-side controller that just calls the server-side star/unstar APIs. Those two functions are same except for requesting methods("put" and "delete").
Also, add "isMyStart()" function to see whether the current user starred a message or not.

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
    }
```

#### Edit HTML template file

Edit the HTML file to add star icons, and call "starThing()" to star on click. If the message is already starred, call "unstarThing()" to unstar the message.


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

Until now, the message list shows all users' messages. Let's add a feature to list only user messages or only starred messages.

#### Add client-side routing
Create new URLs to show each user's messages or starred messages.

* User messages: /users/USER-ID
* Starred messages: /users/USER-ID/starred

Client-side routing is set using "$stateProvider.state" function. Add the above URLs to routing with the same controller ("MainCtrl") and template ("main.html"). To filter messages, we set a query. Add "query" to "resolve" field. Filter by user for user messages, and filter by user ID of "stars" field for starred messages.

On MongoDB, we can write queries using JavaScript. So, we can just transfer the query to MongoDB through the server-side API to filter messages.

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

Add a query parameter to the server-API request. Add "query" to the controller function, and add the "query" to "$http.get()" argument.

client/app/main/main.controller.js:

```javascript
  .controller('MainCtrl', function ($scope, $http, socket, Auth, query) {
  ...
    $http.get('/api/things', {params: {query: query}}).success(function(awesomeThings) {
```

#### Edit server-side controller

On the server-side controller, just transfer the received query to MongoDB by passing the query as "find()" arguments.

server/api/thing/thing.controller.js

```javascript
exports.index = ...
  var query = req.query.query && JSON.parse(req.query.query);
  Thing.find(query).sort...
```


#### Add Nav links to Navbar

Until now, Navbar has only one link "Home". Change it to three links like "All", "Mine", and "Starred".

Add link items to $scope.menu array. Enable "Mine" or "Starred" links only for logged-in users. To switch links dynamically before or after login, set the "link" field(for URL) and the "show" field as functions.

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

On the Navbar HTML file, change from the "link" variable to the "link()" function. Set "item.show()" on "ng-show"  to display only when show() returns true.

client/components/navbar/navbar.html:

```html
        <li ng-repeat="item in menu" ng-class="{active: isActive(item.link())}" ng-show="item.show()">
            <a ng-href="{{item.link()}}">{{item.title}}</a>
        </li>

```

#### Edit client-side HTML template
Show the user message on click user name. Just add a link to the user message URL (/users/userID).

client/app/main/main.html:

```html
        <a ng-href="/users/{{thing.user._id}}">{{thing.user.name}}</a>
```

#### Edit test code
Edit the test code to add a dummy query parameter.

client/app/main/main.controller.spec.js:

```javascript
    MainCtrl = $controller('MainCtrl', {
      $scope: scope,
      query: null,
    });
```

Now, we can see my or other users' messages or starred messages.


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

On routing configuration, write to "url" field like "XXX?keyword" so that we can use "keyword" as a parameter.

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

Add a search box to the Navbar. Set "search(keyword)" to "ng-submit" attribute so that submitting keyword invoke the search function.


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

On the Navbar controller, add a search function to change the URL state to have the searching keyword specified.

client/components/navbar/navbar.controller.js:


```javascript
    $scope.search = function(keyword) {
      $state.go('main', {keyword: keyword});        
    };
```

This works. But, it always searches all messages. It would be nice if we could also restrict the search to user messages or starred messages. Change the "search()" function to keep the URL state (main, user, or starred) on search. Don't forget to add '$state' to the NavbarCtrl function argument to use the variable.

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

Regular expression search works, but it will become slow if we have many messages.

So, let's use full-text search MongoDB provides. MongoDB have '$text' / '$search' operators for full text search. For full-text search, we don't (can't) specify field to search.



```javascript
    var keyword = $location.search().keyword;
    if(keyword){
      query = _.merge(query, {$text: {$search: keyword}});
    }
    $http.get('/api/things', {params: {query: query}})...
```

#### Edit server-side model

For the full-text search, add a 'text' index to the searching field on the schema.

server/api/thing/thing.mode.js:

```javascript
ThingSchema.index({name: 'text'});
```

Now, we can search by word like "Development". (We cannot search by substring match.)


####  Japanese search
MongoDB's full-text searching only supports Latin languages, it and does not support other languages as Japanese.

We can use ElasticSearch or other engines. But for now, we will use "TinySegmenter" to tokenize Japanese.

Add "tokenizedName" field to store and index tokenized messages.

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

Tokenize a message on save, and join with space, and save to the "tokenizedName" field. You can hook on save by calling "pre('save', callback)" on the schema.

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

Now, we can search by Japanese words. For example, we can search the message "吾輩は猫である" for the word "我輩".

<div id="infinite_scroll"></div>
Infinite scroll
============

Now, we can only see the last 20 messages, and there is no way to see older messages.

Let's add an infinite scroll to see older messages, like Twitter does.

#### On client-side, install "ngInfiniteScroll" library
Install an AngularJS library "ngInfiniteScroll" for an infinite scroll.

```javascript
% bower install --save ngInfiniteScroll
% grunt wiredep
```

#### Load ngInfiniteScroll
To use the ngInfiniteScroll module, add it to the AnguarJS application module dependency.

client/app/app.js:

```javascript
angular.module('paizatterApp', [
   ... ,
   'infinite-scroll'
]);
```

#### Edit HTML file

To use the ngInfiniteScroll module, on div tag of "container" class, add "infinite-scroll" attribute to call a function ("nextPage()") on scroll. Set flags ("busy", "noMoreData") to "infinite-scroll-disabled" attribute not to scroll while loading or if no more messages are available.

At the end of the HTML file, output "Loading data" while loading.

client/app/main/main.html:

```html
<div class="container" infinite-scroll='nextPage()' infinite-scroll-disabled='busy || noMoreData'>
  ...
  <div ng-show='busy'>Loading data...</div>
</div>
```

#### Edit client-side controller

On the "$scope" variable, create "busy" field to store for the loading state, and "noMoreData" flag to store whether all the message is loaded or not.

On scroll, we need to load messages older than the last message. Add "{_id: {$lt: lastId}}" to the query. 

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

Now, we can load messages older than the last 20 messages, using infinite scroll.

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

```shell
% grunt test
```

<div id="re_deploy"></div>
Re-deploy
===============

Finally, we have built all the features!
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
In this article, we created a Twitter-like full-stack web service using a MEAN stack, Angular Full-Stack generator. 
Although we have not edited many lines of codes, we have build a nearly full-fledged, real web service.

With MEAN stack, we can easily create web services just using JavaScript.
Let's come up with ideas and build your own web services!

I welcome your feedback about the instruction.

I'll continue writing articles about creating web services using MEAN stack.
