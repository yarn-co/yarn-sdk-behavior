# APIBehavior
APIBehavior makes it super easy to interface with APIs in Polymer!

install with `bower install yarn-api-behavior`

This behavior is intended to be used to create a Polymer element to interface with an API you want to use in your project.


## Configuring Endpoints
* Create an element, name it something like \<my-api\>.

* In your api element, you configure endpoints in an "endpoints" property like so:
```javascript

ready: function() {
  this.set('endpoints', {
    getFakeData: {
      protocol: 'http',
      host: 'localhost:3001',
      method: 'GET',
      path: '/get/some/local/json/file.json'
    },
    getRealData: {
      host: 'websiteWithRealData.com',
      method: 'GET',
      path: '/path/to/endpoint'
    },
    // httpbin will send back what you post, you can test APIBehavior with that endpoint
    postSomething: {
      protocol: 'http',
      host: 'httpbin.org',
      method: 'POST',
      path: '/post'
    }
  });
}

/*
  Each endpoint is required to have these properties:
  method,
  path

  Optional properties for each endpoint are:
  protocol,   // default is http
  host,       // default is localhost:3001
  handleAs    // default is json
*/

/*
  There are properties api-behavior gives you that lets
  you set defaults for the host or protocol options.
  These values will be used in an endpoint when none
  are explicitly configured:
  
  defaultProtocol,
  defaultHost
*/
```
---

##Hook methods
There are two hook methods for ajax requests. One before the call, and one after it returns.
You can set these to functions in your api element like so:
```javascript


ready: function() {
  this.set('apiPreRequestHook', function(body, params, cb) {
    // body is null if it's a GET request
    // Do things you want done before each call. You get info from the request.
  });
  
  this.set('apiResponseHook', function(err, req, requestCb, continueCb) {
    /*
      The var I named requestCb is either 
      a function callback or an object with properties "resolve" and "reject",
      which are effectively linked to your code where you made the api call.
    */
    
    /*
      requestCb is something optional, you can directly call your callbacks in your
      code, 
      or let them get called on their own by calling continueCb(); and not using requestCb.
      
      If you use requestCb, ignore continueCb.
      If you use continueCb, ignore requestCb.
      
      If you'd like better naming for these things, I'm totally open to changing the examples.
    */
    
    if(err) {
      if(requestCb.reject) {
        requestCb.reject(err);
      } else {
        requestCb(err, null);
      }
    } else {
      if(requestCb.resolve) {
        requestCb.resolve(req)
      } else {
        requestCb(null, req);
      }
    }
    
    // requestCb(null, req);
    // continueCb();
    
    // Do things you want done after each call.
    // This gets called before the callback you define.
    // I think you can mutate the req object before the callback gets it, haven't tried it yet though.
  });
}
```

## Headers
If you want to set headers for your endpoints, you'll have to write a function in your api element that sets the 'headers' property that api-behavior exposes. Call this function right before calling your endpoint, it'll set the headers, then make your call with those headers. Right now you'll have to set the headers before each call you make to ensure the headers aren't the ones intended for the most recent call you made, whether you want them to have something or be set to default {}
Example:
```javascript
setHeadersForNextCall: function(headersObj) {
  this.set('headers', headersObj);
}
```

## Making calls to endpoints

* For each endpoint configured, api-behavior generates a function on your api element with the same name you gave the respective endpoint.

* In another element, you'll want to import your \<my-api\> element and stick it in the template. Give it an id, say, "myApi".

* Now make calls to your endpoints like this:

```javascript
attached: function() {
  this.$.myApi.getFakeData(function(err, request){
    if(err) {
      console.log(err);
    } else {
      console.log(request.xhr.response);
    }
  });
}
```

If your endpoint method was GET, then the function should have these arguments:
```javascript
// We support function overloading
myApiElement.myEndpointName(parameters, callback);
myApiElement.myEndpointName(callback);
```
If your endpoint method was POST, PUT, or DELETE, the function should have these arguments:
```javascript
// Function overloading
myApiElement.myEndpointName(body, parameters, callback);
myApiElement.myEndpointName(body, callback);
```

The callback will receive a request object, which has an xhr property, and api-behavior assumes you'll be receiving JSON from the server.
If you want the response to be a different type, then configure the expected response type in a 'handleAs' property for that endpoint.

## Support for Promises!
You can optionally receive a promise returned from your api function, use it like this:
```javascript
// For a post
myApiElement.myEndpointName(body).then(function(request){
  console.log(request.xhr.response);
})
.catch(function(reason) {
  console.log(reason);
});
```

## Error handling
If an ajax call returns and (request.xhr.status >= 400 && request.xhr.status < 600), an 'api-error' event will be fired. It will pass an object with the event, which will be in e.detail in your event handler.
Inside e.detail, the object will have these properties:
```javascript
{
  api: //the api element this behavior belongs to,
  response: //the response
}
```

## More examples
```javascript
attached: function() {

  this.$.myApi.setHeaders({Authorization: "JSONWebToken--lajjfkljdLKSFIf28283rji93..."});
  this.$.myApi.getRealData({
    urlParam1: "aValue1",
    urlParam2: "aValue2"
  }).then(function(req){
    console.log(req.xhr.response);
  }).catch(function(err) {
    console.log(err);
  });
  
  this.$.myApi.setHeaders({});
  this.$.myApi.postSomething({
    aKey: "aValue"
  },
  function(err, req) {
    if(err) {
      console.log(err);
    } else {
      console.log(req);
    }
  });
}
```


That's it! =)
