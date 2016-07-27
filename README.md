Webix Remote - Nodejs
---------------------


[![Build Status](https://travis-ci.org/webix-hub/webix-remote-js.svg?branch=master)](https://travis-ci.org/webix-hub/webix-remote-js)
[![npm version](https://badge.fury.io/js/webix-remote.svg)](https://badge.fury.io/js/webix-remote)
[![Join the chat at https://gitter.im/webix-hub/webix](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/webix-hub/webix) 


Simple RPC for NodeJS and Webix UI


### How to install

To install Webix Remote, simply run the command below:

```
npm install webix-remote
```

### How to use

#### On the server side

Include the "webix-remote" module and create a server with the help of the corresponding method:

```js
var remote = require("webix-remote");
var api = remote.server();
```
After that you can use server-side API, e.g. use *setMethod()* to register methods on the server side and then refer to them from the client side.
The method takes two parameters:

- name - (string) the name under which the method/methods will be registered
- function - (function/object) the method/methods registered under the specified "name"

In the example below setMethod() is used:

- to set the *add* function that will sum up two numbers 
- to specify the *helpers* object that contains a set of functions:

```js
api.setMethod("add", function(a,b){
	return a+b;
});

api.setMethod("helpers", {
	add: (a,b) => a+b,
	mul: (a,b) => a*b
});

// in case you want to use the express framework
express.use("/api", api.express());
```
Now you will be able to refer to the registered methods from the client side.

#### On the client side

Include the path to the server-side API after the *webix.js* file:

```html
<script src='webix.js'></script>
<script src='/api'></script>
```

Then to call a server-side method you need to use the *webix.remote.methodName* call.

You can ask the *add* function registered on the server to return the sum of two numbers. 
The result of the operation will be put into the "result" variable:

```js
var result = webix.remote.add(1,2);
//or using the then() method
webix.remote.add(1,2).then(result){
    alert(result);
});
```
To render page without delays, Webix Remote loads data asynchronously. So the "result" variable will get a promise of data, while 
the real data will come later.

You can go the synchronous way and and load data using the *sync()* method:

```js
var result = webix.remote.add(1,2).sync();
```

#### Special parameters

To pass extra parameters to the server function, use the *$req* parameter - request object. 
For example, the "add" function can pass to the client a request object with parameters about the current user session.

```js
api.setMethod("add", function(a,b,$req){
	return a+b+$req.session.userBonus;
});
```

The client-side code can be as follows:

```js
webix.remote.add(1,2);
```

#### Adding static data

You can define some static data which will be available on the client side. It is a good place for session data, 
which should be shared with the client-side code.

To specify static data, you need to use the setData() method and pass two arguments to it:

- name - (string) the name of parameter that will be set as static
- handler - (function) the function that will take the request object and return the result

Let's set the "$user" parameter on the server side. The function specified as the second parameter will get a request object and return the user id:

```js
api.setData("$user", function($req){
	return $req.user.id;
});
```
Instead of function you can pass some value as a second parameter:

```js
api.setData("$user",1);
```
But be careful, in this case the data generation method will be called only once, during the API initialization. 

As for the client side, you can get user data in the following way:

```js
var user = webix.remote.$user;
```

#### API access levels

With Webix Remote, you can limit access to particular methods by the user's access level - a role assigned to him/her. 

You need to specify the role that allows using the method during the method's creation like this:

```js
//only user with 'admin' role will be able to call the method
api.setMethod("admin@add", function(a,b){
	return a+b;
});
```

Access levels are defined by the access modifier specified with the *req.session* object:

- all methods without an access modifier are allowed by default
- if "req.session.user" exists, methods with the *user* modifier are allowed
- if "req.session.user.role" exists, methods with a particular role modifier are allowed

For example, the following access modifier is set:

```js
req.session = { user: { role:"admin,levelB"}}
```

The *add* function will be allowed for users with the "user", "user.role=admin" and "user.role=levelB" access modifiers. 
For users with any other role the method will be unavailable:

```js
api.setMethod("user@add1", (a,b) => a+b ); //allowed
api.setMethod("admin@add2", (a,b) => a+b ); //allowed
api.setMethod("levelC@add3", (a,b) => a+b ); //blocked
```
####Custom Logic for Access Levels

You can define one complex function containing a set of access level rules by using the *$access* parameter. For example:

```js
api.$access = function(){
	return { 
		user : !!req.user,
		admin: req.user && req.user.id == 1 
	};
};
```
The above function will check whether a user exists and whether he/she possesses the admin role at the same time.


### License

MIT
