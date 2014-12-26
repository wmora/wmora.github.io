--- 
title: Angular Espresso - AngularJS, CoffeeScript and Express
date: "2013-04-01T22:29:00.001-03:00"
author: William Mora
tags: 
- socket.io
- angularjs
- angular-espresso
- coffeescript
- node.js
- git
- github
- mongoose
- express
- mocha
permalink: /2013/04/angular-espresso-angularjs-coffeescript.html
---

_**UPDATE:** This post is now outdated. Check out the project's [git page](http://wmora.github.io/angular-espresso/) for the latest info on Angular Espresso._

[AngularJS](http://angularjs.org/) is a powerful framework maintained by Google. I started playing around with it a few weeks ago and I think it is great for developing web apps. If you are a used to using [jQuery](http://jquery.com/) for DOM manipulation and/or your views' logic, you'll have to change your approach and change the way you write dynamic views. Best of all, AngularJS is more than that: you could also have models, controllers, services and use partial views to develop a true, behavior-driven web app.

The AngularJS team has an [angular-seed](https://github.com/angular/angular-seed) project that is very useful as a starting point for an AnglarJS app. There's also an [extension](https://github.com/btford/angular-express-seed) of that project that integrates [Express](http://expressjs.com/) with AngularJS.
<!--more-->

Both of those projects are great. Based on them, I created a starting point for an AngularJS project, powered by [Node.js](http://nodejs.org/) and Express at the backend, all written in [CoffeeScript](http://coffeescript.org/). The result is [Angular Espresso](http://wmora.github.com/angular-espresso/).

Besides rewriting some of the angular-seed project in CoffeeScript, I included a more complete template that hopefully serves as a starting point for some powerful web apps. Some of its key points are:

*   Socket.IO: [Socket.IO](http://socket.io/) is included as a service. As it is, it implements the "on" and "emit" functions.
*   Web Service: The "User" service is an example of how the project should be developed. Call a web service from your client app to your Express server and let the server handle all the logic, even if it's just a proxy to an external service.
*   Partial Views and Jade: Use [Jade](http://jade-lang.com/) as the template engine for your views and partial views. In your server, define the top-level views and let AngularJS handle the rest of the views with a `$routeProvider` config. 

Once you download the project, you'll need to install the dependencies with npm and build/run the project using its Cakefile. Take a look at the [project's documentation](http://wmora.github.com/angular-espresso/) for more info. Long story short, the `app` folder contains all `.coffee` files. Each folder has a README file with a brief description. It compiles into a module called `.espresso` that it is run by the top-level `app` file. All client `.js` files are uglified using [UglifyJS](https://github.com/mishoo/UglifyJS2).

In the future I'd like to add some kind of Model support for the server ([Mongoose](http://mongoosejs.com/), maybe?) as well as testing support (probably [Mocha](http://visionmedia.github.com/mocha/)).