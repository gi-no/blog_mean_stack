<!-- Title: Building full-stack web service - MEAN stack development(1) -->
<!-- URL: http://engineering.paiza.io/entry/2015/07/08/153011 -->

[f:id:paiza:20140712194904j:plain] (by Yoshioka Tsuneo, [twitter:@yoshiokatsuneo] at https://paiza.IO/)

(Japanese article is available <a href="http://paiza.hatenablog.com/entry/2015/07/08/最新・最速！Webサービスが今すぐ作れる！_-_MEANスタッ">here</a>.)

Nowadays, it is getting hard to build web service because we need to use full-stack environment: browser(client) code for interactive UI, server code for shared data or logic, and websockets for realtime communication.

MEAN stack package frameworks from client side to server side, and make web service development much simpler, easier, and faster.

In this article, I introduce one of MEAN stack implementation, AngularJS Full-stack generator(generator-angular-fullstack), and actually run, edit and deploy application.

There are many articles about development only for client, or only for server. But, not so many mentions how to mash up those individual tools. Thanks to MEAN stack, now we have best practice to build full stack application without bothering mashing up things. Let's try for the cool environment !

In the following articles, based on this article, I'll also introduce the way to build more practical applications.



[f:id:paiza:20150703174202p:plain]

[AngularJS Full-stack generator Demo:http://fullstack-demo.herokuapp.com](http://fullstack-demo.herokuapp.com)

Contents
=================
* [What is MEAN stack ?](#about)
* [MEAN stack features](#features)
* [Install](#install)
* [Create a new project](#new_project)
* [Start server](#run)
* [Edit code](#edit)
* [Debug](#debug)
* [Deployment](#deploy)
* [SNS integration](#sns_link)
* [Summary](#summary)
* [Reference](#reference)

<div id="about"></div>
What is MEAN stack ?
=======================

MEAN stack is a package of frameworks, MongoDB, Express, AngularJS, and Node.js, to build full-stack web service.

[https://www.mongodb.org:image=https://upload.wikimedia.org/wikipedia/commons/e/eb/MongoDB_Logo.png]

[http://expressjs.com/:image=https://upload.wikimedia.org/wikipedia/commons/6/64/Expressjs.png]

[https://angularjs.org:image=https://raw.githubusercontent.com/angular/angular.js/master/images/logo/AngularJS.exports/AngularJS-medium.png]

[https://nodejs.org/:image=https://nodejs.org/images/logos/nodejs.png]


In the past, there are web service frameworks like CakePHP, Ruby on Rails, that provides best practices to build web service easily, quickly, and securely.

Frameworks like CakePHP or Ruby on Rails is based on the basic HTTP flow, client send requests to servers, servers generate and send HTML page to client, and client render the HTML on browser.
Form action or following links completely discards current page and render new pages generated on the servers.

But, these request response based architecture causes delays on every action, and clear all states(inputs, selection scrolling, etc...), and does not provides interactive UI.

So, to make web more interactive, web services like Google Maps starts using JavaScript based Ajax technology.

At first, JavaScript is used only on specific UI parts. But, more and more parts are built using JavaScript, and eventually JavaScript framework is demanded to use much JavaScript code build web application.
AngularJS is one of the most widely used such a (client side) JavaScript framework.

Furthermore, request-response style architecture prevent web services to actively send notification to users. There are little way to send new message notification on chat application, or display other player's move on the multiplayer game.
WebSocket is a technology to solve such a restriction on the web with full-duplex persistent connection. On WebSocket, because browser and server application are connected persistently, server application need to handle bunch of connections at the same time.
Node.js  manage multiple clients(browsers) on a single thread utilizing asynchronous operations JavaScript have.

So now, we can use JavaScript for both client side, and server side. Now, it is time to introduce JavaScript database MongoDB. MongoDB is schema less, so is best fit for rapid development.

MEAN stack is such a JavaScript based full-stack environment combining client side framework AngularJS, server side framework Node.js/Express.JS, and database MongoDB.

MEAN stack itself is a name for combination. Actual implementation includes MEAN.IO or MEAN.JS.
In this article, I introduce AngularJS Full-stack generator(generator-angular-stack) that is easy to start with sophisticated generators and templates.


<div id="features"></div>
MEAN stack features(generator-angular-fullstack)
=======================

#### Full-stack

Full-stack environments provides a best practice for client, server and database. That makes easy to understand whole project structure. Especially, it is quite convenient to have pre-configured combination among client, server, database, etc... 


#### Only one language: JavaScript
We can use one language, JavaScript, to develop whole projects including client, server and database.
It is quite stress-free that there is no need to switch context among languages.

#### Based on common tools
Frameworks used in MEAN stack, MongoDB, Node.js, Express, or AngularJS, are not only for MEAN stack. Individual frameworks are just widely used.
Tools used in AngularJS Full-stack generator, Yeoman, Bower, npm, Grunt, Karma are also widely used tools.

Those common frameworks or tools introduce develop environment stableness, use cases, knowledges. Web development environment changes time to time, so it is important to use common tools to be flexible and catch up latest technologies.

#### Generator
AngularJS Full-stack generator can generate project template using menu style interface.
AngularJS Full-stack generator supports following configurations.

* Client
  * Scripts: JavaScript, CoffeeScript, Babel
  * Markup: HTML, Jade
  * Stylesheets: CSS, Stylus, Sass, Less
  * AnguarJS Routers: ngRoute, ui-router

* Server
  * Database: None, MongoDB
  * Authentication boilerplate: Yes, No
  * oAuth integrations: Facebook, Twitter, Google
  * Socket.io integrations: Yes, No

AngularJS Full-Stack generator also provides generators for client-side and server-side code modules.
Generators make it easy to start development. Heroku or OpenShift deployment commands are also provided to release products easily.

<div id="install"></div>
Install
=========================
So, let's install MEAN stack implementation, AngularJS Full-Stack generator.
MEAN stack works on Mac OS X, Linux or Windows. In this article, I'm using Mac OS X, but most commands just works on other OS.

* Install Node.js (If not installed)

Browse https://nodejs.org, and click "Install" to download, then install Node.js package.

* Install Yeoman, Bower, Grunt, Gulp (If not installed)

```sh
% sudo npm install -g yo bower grunt-cli gulp
```

* Install AngularJS Full-Stack generator(generator-angular-fullstack)

```sh
% sudo npm install -g generator-angular-fullstack
```

* Install Homebew (If not installed)

```sh
% ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

* Install MongoDB(If not installed)

```sh
% brew update
% brew install mongodb
% ln -fs /usr/local/opt/mongodb/homebrew.mxcl.mongodb.plist ~/Library/LaunchAgents/
launchctl load ~/Library/LaunchAgents/homebrew.mxcl.mongodb.plist
```

* Test MongoDB

MongoDB has a flexible schema, and does not have "create" statement to declare schema but you can just insert data to database.

In MongoDB, each data("record" in SQL) is called document that stores any JSON objects, and the collection of data("table" in SQL) is just called "collection".

So now, let's try to insert, find or delete documents.

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

There is also GUI tools like [MongoHub](http://mongohub.todayclose.com) on Mac OS X to make database operation easy.

[f:id:paiza:20150706155509p:plain]

[MongoHub](http://mongohub.todayclose.com)

<div id="new_project"></div>
Create a new project
=================
Now, let's create a new project.

* Make project directory

```sh
% mkdir sample
% cd sample
```

* Generate project files using AngularJS Full-Stack generator

```sh
% yo angular-fullstack sample
```

There are questions.
You can just choose defaults, but maybe nice to enable oAuth integration. Let's enable all social networks for now.

```
- Would you like to include additional oAuth strategies? 
 ◉ Google
 ◉ Facebook
❯◯ Twitter

```

Just after answering all questions, project files are generated, and modules are installed. It may take some time for the first time. Just have a cup of coffee.

* Updating npm packages
Default npm packages on Angular Full-stack generator is a bit old, so let's update to latest using npm-check-updates .

```shell
% npm install -g nom-check-updates
% npm-check-updates -u
% npm install
```

<div id="run"></div>
Start server
==========

Let's start server. Just type a command.

```sh
% grunt serve
```

It setup all tasks, and start Node.js server, and opens browser with URL: http://localhost:9000 .

In this point, following features are already implemented.

* Twitter Bootstrap based UI
* Listing, adding, and removing items.
* Real-time listing update on other user's adding or removing items.
* User authentication(Sign Up, Login, SNS integration(with API key))

Let's put some words in the input box and click "Add New", then the item listing is updated. If you opens multiple browsers and add item, other browsers listing is automatically updated without manually reloading browser.

<div id="edit"></div>
Edit code 
================
Let's change file contents. (Keeping "grunt serve" running)

app/main/main.html:

```html
<h1>How's it going ?</h1>
```
Just after saving file, (without manually reloading browser) contents on the browser is updated in real time.

<div id="debug"></div>
Debug
=================

#### Client-side debugging

You can use development tool that web browses have.

On Chrome, click Chrome menu icon right of URL base, choose [More tools][Developer tools], then you'll get "Developer Tools" window.

[f:id:paiza:20150708140450p:plain]

On Safari, go Safari menu's Preference, check "Show development menu" to enable development menu. Open a page to debug, go to main menu and choose "Development" - "Show error console" to get development tools.

On development tools, you can see console logs, set breakpoints on JavaScript code, or print variables.


#### Server-side debugging

Let's try Node.js debugger(Node Inspector) for server side code.
Node Inspector runs on Chrome. So, update "open" function on "Gruntfile.js" like below to specify browser application as "Google Chrome.app".

Gruntfile.js:

```javascript
            // opens browser on initial server start
            nodemon.on('config:update', function () {
              setTimeout(function () {
                require('open')('http://localhost:8080/debug?port=5858', '/Applications/Google Chrome.app');
              }, 500);
            });
```

The, run Node.js server on debug mode.

```sh
% grunt serve:debug
```

Now, chrome developer tools starts to debug Node.js application. The debugger suspended on the first line of the Node.js application. Just click "Resume" button(or "F8") to restart application.

In this window, you can set breakpoint, step in/over, or see variables.

[f:id:paiza:20150706114630p:plain]

<div id="deploy"></div>
Deployment
===========
Because "grunt serve" starts server on the local machine, other users cannot use your web service. To make it public, deploy application on the public server.

Angular Full-Stack generator supports Heroku deployment.

#### Create Heroku account
Heroku provides free plan to deploy one application. Just browse Heroku and click "Sign up for free" to create new account.

https://www.heroku.com

[f:id:paiza:20150706114938p:plain]

#### Install Heroku Toolbelt

To manage application on Heroku, you will use Heroku Toolbelt command line toolkit. Download and install from URL below.

https://toolbelt.heroku.com

Let's login using "heroku" command.

```sh
% heroku login
```

Type username and password for heroku.

#### Set up application deployment environment for Heroku

To set up application deployment environment for Heroku, use Angular Full-stack generator command.

"yo angular-fullstack:heroku" command set up Heroku deployment environment. Type application name asked. The application URL will be http://APPLICATION.herokuapp.com .

You also need to install MongoDB module. Heroku support MongoHQ and MongoLab. For now, let's use MongoLab that have free plan.

```sh
% yo angular-fullstack:heroku
% cd dist
% heroku addons:add mongolab
```

From now on, you can build and deploy application using "grunt" and "grunt buildcontrol:heroku" command.

```sh
% grunt
% grunt buildcontrol:heroku
```

Done !

Let's access application URL with browser.

http://APPLICATION.herokuapp.com/

In case you got error, check out log messages.
```sh
% heroku logs
```

<div id="sns_link"></div>
SNS integration
==========
Although Sign Up and Login is already implemented, you need to set up API key and SECRET key to integrate SNS authentication.

#### Twitter

  On https://apps.twitter.com , click "Create New App" to create application. Specify application URL on "Callback URL"(Ex: http://paizatter.herokuapp.com ).

On application page, see "Keys and Access Tokens" to get "Consumer Key (API Key)" and "Consumer Secret (API Secret)".

[f:id:paiza:20150706115258p:plain]

#### Facebook

  On https://developers.facebook.com/apps/ , click "Add a New App". Choose "Website" and input application name and choose category like "Utilities" on "Choose a Category", then click "Create App ID" to create.

  Choose you application from "My Apps" menu. Choose "Advanced" tab and specify application URL on "Valid OAuth redirect URIs" (Ex: http://アプリケーション名.herokuapp.com ). You can specify multiple URL, so adding "http://localhost:9000/" will be convenient to test on local environment.

[f:id:paiza:20150706115559p:plain]

From application's "Settings" menu, input "Contact Email". Choose "Status & Review" and answer "Yes" on "Do you want to make this app and all its live features available to the general public?" to enable application.
Now, you can get access Dashboard to get "App ID" and "App Secret".

#### Google
  On https://console.developers.google.com/project , click "Create project".
Choose created project. From menu on left side, choose "APIs&auth"/"Credentials". On OAuth page, click "Create new Client ID" and choose "Web application" and click "Configure consent screen".

After the consent, put application URL on "Authorized redirect URIs" (Ex: http://APPLICATION.herokuapp.com/auth/google/callback ) and click "Create Client ID"

[f:id:paiza:20150706124704p:plain]

From "APIs&auth"-"API" menu, choose "Google+ API" and enable "Google+ API".
Now, you can see "APIs&auth"/"Credentials" to get Client ID(API key) and Client secret(SECRET key).

#### Set up API key on Heroku

After getting API keys and SECRET keys, use "heroku config set" command to set keys on environment variables.

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
Summary
===========
In this article, I introduced AngularJS Full-Stack generator to build, run and deploy MEAN stack web service. MEAN stack make it easy to build full-stack web service quickly. So, it is especially convenient for project startup, or prototyping. Let's try it !

This article just run sample application. Following article will provides more practical application.


<div id="references"></div>
Reference
=========
AngularJS Full-Stack generator

[https://github.com/DaftMonk/generator-angular-fullstack]



<!--/div-->