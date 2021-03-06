Rainbow
=======

[简体中文](http://mytharcher.github.com/posts/npm-rainbow.html)

A node [Express][] router middleware for Ajax RESTful API base on certain folder path.

Rainbow mapping all HTTP request route to controllers folder each as path to file as URL.

## Installation ##

```bash
$ npm install rainbow
```

## Usage ##

In your express application main file `app.js`:

```javascript
var express = require('express');
var rainbow = require('rainbow');

var app = express();

// Here using Rainbow to initialize all routers
rainbow.route(app);

app.listen(6060);
```

### Controllers ###

All your controllers for catching HTTP request should be defined in each file in `controllers/` folder (could be changed) as same path in URL.

This is the core design for Rainbow! And it makes routing much simpler only by files' paths!

Here writes a router `something.js` in your `controllers/` folder like this:

```javascript
exports.GET = function (req, res) {
	res.send(200, 'Simple getting.');
};
```

If you need some filters, just add a `filters` array property which contains your filters in `filters/` folder to the handle function like this:

```javascript
exports.GET = function (req, res) {
	res.send(200, 'Simple getting.');
};
// add filters
exports.GET.filters = ['authorization'];
```

Also you could define other HTTP methods handlers, but make sure in one file each URL! Example in `controllers/user.js`:

```javascript
exports.GET = function (req, res) {
	User.find({where: req.query.name}).success(function (user) {
		res.send(200, user);
	});
};

exports.PUT = function (req, res) {
	User.create(req.body).success(function (user) {
		res.send(201, user.id);
	});
};

// You can also define `post` and `delete` handlers.
// ...
```

If you want all methods to be process in only one controller(something not RESTful), just make exports to be the handle function:

```javascript
module.exports = function (req, res) {
	// all your process
};
```

You can write controllers with coffeescript using `.coffee` in example `controllers/user.coffee`:

```coffeescript
exports.GET = (req, res) ->
	User.find(where: req.query.name)
	.success (user) ->
		res.send 200, user

exports.PUT = (req, res) ->
	User.create(req.body)
	.success (user) ->
		res.send 201, user.id
```

### Params ###

Rainbow started to support param form URL from version 0.1.0. Now you can define your controllers URL with params resolved by native Express like this:

```javascript
exports.GET = function (req, res) {
	var id = req.params.id;
	// your business
};

exports.GET.params = ':id?';
```

Or you can use regular expression also:

```javascript
exports.GET = function (req, res) {
	console.log(req.params);
}

exports.GET.params = /(\d+)(?:\.\.(\d+))?/;
```

But make sure no regular expression `^` used as starter and `$` as ender, or rainbow could not resolve the expression correctly.

### Filters ###

Make sure the filters you need had been defined in `filters/` folder (could be changed) as same module name, because them will be required when initilizing. Here `authorization.js` is a example for intecepting by non-authenticated user before `GET` `http://yourapp:6060/something`:

```javascript
module.exports = function (req, res, next) {
	console.log('processing authorization...');
	var session = req.session;
	
	if (session.userId) {
		console.log('user(%d) in session', session.userId);
		next();
	} else {
		console.log('out of session');
		// Async filter is ok with express!
		db.User.find().success(function (user) {
			if (!user) {
				res.send(403);
				res.end();
			}
		});
	}
};
```

You could see filters is as same as a origin router in Express, just be put together in `filters/` folder to be interceptors like in Java SSH.

Filters also support variable name of filter function in same file than string filter file name in filters directory (from v0.4):

```javascript
// controller file test.js route to [GET]/test
function myFilter (req, res, next) {
	// blablabla...
	next();
}

exports.GET = function (req, res) {
	// blablabla...
};

exports.GET.filters = [myFilter];
```

If you need some filters to be applied for all methods in an URL, you could use URL level filters definition:

```javascript
// controller file test.js route to [GET|POST]/test
exports.GET = function (req, res) {};
exports.POST = function (req, res) {};
exports.POST.filters = ['validation'];
exports.filters = ['session'];
```

When user `GET:/test` the filter `session` would run, and when `POST:/test` URL level filter `session` run first and then `validation`.

### Change default path ###

Controllers and filters default path could be changed by passing a path config object to `route` function when initializing:

```javascript
rainbow.route(app, {
	controllers: '/your/controllers/path',
	filters: '/your/filters/path'
});
```

These paths are all **RELATIVE** to your app path!

## Troubleshooting ##

0. Gmail me: mytharcher
0. Write a [issue](https://github.com/mytharcher/rainbow/issues)
0. Send your pull request to me.

## MIT Licensed ##

-EOF-

[Express]: http://expressjs.com/
