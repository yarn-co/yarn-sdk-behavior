# SDKBehavior
SDKBehavior makes it super easy to use your APIs in Polymer!

install with `bower install yarn-sdk-behavior`

This behavior is a tool to help create sdk elements that interface with organized groups of API endpoints you want to use in your project.

Sdk behavior allows you to make api calls like this:

(From here on out, the functions in the example below will be referred to as your **business logic api calls**)
```
<link rel="import" href="../path/to/element/my-sdk/my-sdk.html">

<!-- This my-sdk element will have configuration to make the below call possible
     and tearless when using in your business logic. -->

<my-sdk id="mySdk"></my-sdk>

...

ready: function() {
  
  // Callbacky
  this.$.mySdk.callApiEndpoint(function(err, req) {
    
  });
  
  // Promisey
  this.$.mySdk.callApiEndpoint().then(function(req) {
    //req is an iron-request object. The good stuff is in req.xhr.response
  })
  .catch(function(err) {
    
  });
}
```

## Configuring Endpoints
* Create an element, name it something like \<my-sdk\>.

* In your sdk element, you configure endpoints like so:
* Note: because of some underlying things going on, it's best to configure your endpoints as an object
above the Polymer({}) function call as below
```javascript

(function() {

  var endpoints = {
    getTestData: {
      protocol: 'http',
      host: 'localhost:3000',
      method: 'GET',
      path: '/get/some/local/json/file.json'
    },
    getRealData: {
      host: 'websiteWithRealData.com',
      method: 'GET',
      path: '/path/to/endpoint'
    },
    postSomething: {
      protocol: 'http',
      host: 'httpbin.org',
      method: 'POST',
      path: '/post'
    }
  }
  
  Polymer({
    behaviors: [
      YarnBehaviors.SDKBehavior
    ],
  
    is: 'my-sdk',
    
    properties: {
    },
    
    ready: function() {
      
      // You must set your endpoints to this property
      this.set('endpoints', endpoints);
      
    }
    
    ...
    
  });
})();
```


##### Each endpoint is required to have at least these properties:

* method
* path

#####Optional endpoint properties with yarn-sdk-behavior defined defaults already set are:

* protocol  
  * default is already set to http. Set this to override the default.
* host
  * default is already set to '' empty string. Set this to override the default.
* handleAs
  * default is already set to application/json. Set this to override the default.

#####Optional endpoint properties with no defaults:
* headers
  * can be set to 'default' (you can define what the default is as shown below) or a JSON object like {'Authorization': 'myAuthKey', 'Accept': 'application/json'}
* withCredentials
  * set to true to send cookies up with your request


#####There are properties sdk-behavior gives you that let you set defaults for host, protocol, or header options.

These values will be used in an endpoint when none
are explicitly configured (e.g. setting 'host' on an endpoint will override the defaultHost). Set these if you want all of your endpoints to have these settings except for a couple, which you'll override in the endpoint configuration with 'host' or 'protocol'.
  
* defaultProtocol
* defaultHost
  

##### The exception to this is defaultHeaders, which will NOT get automatically added to your endpoints:
  
* defaultHeaders
  * lets you set a default value for endpoint headers. 
  Used when headers above is set to 'default'
  defaultHeaders is useful if you're using the same authentication key for many endpoints in your sdk element. Again, these will not get set unless you specify headers: 'default' in your endpoint.


---

##Hook methods
There are two hook methods for ajax requests. One before the call goes out, and one after it returns but before the response gets to your business logic callback.
You can set these to functions in your sdk element like so:
```javascript

ready: function() {
  
  // body will be null if the relevant request method is a POST, or PUT, DELETE, null if a GET
  // params will only be set if there were url params passed in from the business logic call.
  this.set('sdkPreRequestHook', function(endpoint, body, params) {
    // ...
  });
  
  
  /*
    Note: the sdkResponseHook acts differently depending on whether your business logic api call is
    used as a function callback or a promise. The 3rd property (here named "businessLogicCallback") will either be a function callback
    or an object with "resolve" and "reject" properties set. These are functions are essentially your
    business logic callbacks. What you pass them will get passed to your business logic.
  */
  
  /*
    "next" is to be used if you want to continue without calling the businessLogicCallback yourself.
    This will allow yarn-sdk-behavior to finish the course and do whatever it was going to do by default.
  */
  
  // Callbacky
  this.set('sdkResponseHook', function(err, req, businessLogicCallback, next) {
    // Could be used to throw custom errors, used for every api endpoint in this sdk file.
    var res = req.xhr.response;
    if(typeof res === "object") {
      businessLogicCallback(null, res);
    } else {
      businessLogicCallback(new Error("Invalid response type, must be an object"), res);
    }
  });
  
  // Promisey
  this.set('sdkResponseHook', function(err, req, businessLogicCallback, defaultFinish) {
    // Could be used to throw custom errors, used for every api endpoint in this sdk file.
    var res = req.xhr.response;
    if(typeof res === "object") {
      businessLogicCallback.resolve(res);
    } else {
      businessLogicCallback.reject(new Error("Invalid response type, must be an object"));
    }
  });
  
  
  // Use next instead
  this.set('sdkResponseHook', function(err, req, businessLogicCallback, defaultFinish) {
    // Could be used to throw custom errors, used for every api endpoint in this sdk file.
    var res = req.xhr.response;
    if(typeof res === "object") {
      defaultFinish();
    } else {
      throw new Error("Invalid response type, must be an object");
      defaultFinish();
    }
  });
}
```


## Making business logic calls to endpoints

* For each endpoint configured, yarn-sdk-behavior generates a function on your sdk element with the same name you gave the respective endpoint.

* In your business element, you'll want to import your \<my-sdk\> element and stick it in the template. Give it an id, say, "mySdk".

* Now make calls to your endpoints like this:

```javascript
attached: function() {
  
  // Callbacky
  this.$.mySdk.getTestData(function(err, req){
    if(err) {
      console.log(err);
    } else {
      console.log(req.xhr.response);
    }
  });
  
  // Promisey
  this.$.mySdk.getTestData().then(function(req) {
    console.log(req.xhr.response);
  })
  .catch(function(err) {
    console.log(err);
  });
}
```

If your endpoint method was GET, then the function should have these arguments:
```javascript

// Support for function overloading
this.$.mySdk.myEndpointName(parameters, callback);
this.$.mySdk.myEndpointName(callback);

// Promisey
this.$.mySdk.myEndpointName(parameters).then....;
this.$.mySdk.myEndpointName().then....;

// example setting url parameter:
this.$.mySdk.myEndpointName({'user': '1'}).then(function(req){
  console.log(req);
});

```
If your endpoint method was POST, PUT, or DELETE, the function should have these arguments:
```javascript
// Function overloading, body is always first
this.$.mySdk.myEndpointName(body, parameters, callback);
this.$.mySdk.myEndpointName(body, callback);

// Promisey
this.$.mySdk.myEndpointName(body, parameters).then....;
this.$.mySdk.myEndpointName(body).then....;

// example:
this.$.mySdk.myEndpointName({'user': '1'}).then(function(req){
  console.log(req);
});

```

The callback will receive a request object, which has an xhr property, and sdk-behavior assumes you'll be receiving JSON from the server.
If you want the response to be a different type, then configure the expected response type in a 'handleAs' property for that endpoint.


## Error handling
If an ajax call returns and the response from the server was 400, or 402-600, an 'sdk-error' event will be fired. It will pass an object with the event, which will be in e.detail in your event handler.
Inside e.detail, the object will have these properties:
```javascript
{
  sdk: //the sdk element this behavior belongs to,
  err: //the error
}
```


## More examples
```javascript

// In mySdk element:
var endpoints = {
  getRealData: {
    method: "GET",
    withCredentials: true,
    path: '/myEndpoint/{urlParam1}/{urlParam2}'
  },
  postData: {
    method: "POST",
    path: '/post/path'
  },
  sendMail: {
    method: "POST",
    headers: {Authorization: 'jsonWebToken--oi2jio2j3rj23rj23908fj092iefm'},
    path: '/send/mail/to/{place}'
  }
}


// In business logic element
attached: function() {
  
  this.$.mySdk.getRealData({
    urlParam1: "aValue1",
    urlParam2: "aValue2"
  }).then(function(req){
    console.log(req.xhr.response);
  }).catch(function(err) {
    console.log(err);
  });
  
  this.$.mySdk.postData({
    aKey: "aValue"
  },
  function(err, req) {
    if(err) {
      console.log(err);
    } else {
      console.log(req);
    }
  });
  
  this.$.mySdk.sendMail({message: "hi mars!"},
  {place: "mars"})
  .then(function(req) {
    console.log(req);
  })
  .catch(function(err) {
    console.log(err);
  });
  
}
```
