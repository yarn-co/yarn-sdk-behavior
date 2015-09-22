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
    // httpbin will send back what you post
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
  this.$.myApi.getFakeData(function(request){
    console.log(request.xhr.response);
  });
}
```

If your endpoint method was GET, then the function can have these arguments--
```javascript
// We support function overloading
myApiElement.myEndpointName(parameters, callback);
myApiElement.myEndpointName(callback);
```
If your endpoint method was POST, PUT, or DELETE, the function will have these arguments--
```javascript
// Function overloading
myApiElement.myEndpointName(body, parameters, callback);
myApiElement.myEndpointName(body, callback);
```

The callback will receive a request object, which has an xhr property, and api-behavior assumes you'll be receiving JSON from the server.
If you want the response to be a different type, then put the response type in a 'handleAs' property for that
endpoint.

## Support for Promises!
You can optionally receive a promise returned from your api function, use it like this:
```javascript
// For a post
myApiElement.myEndpointName(body).then(function(request){
  console.log(request.xhr.response);
})
```



## More examples:
```javascript
attached: function() {

  this.$.myApi.setHeaders({Authorization: "JSONWebToken--lajjfkljdLKSFIf28283rji93..."});
  this.$.myApi.getRealData({
    urlParam1: "aValue1",
    urlParam2: "aValue2"
  }).then(function(req){
    console.log(req.xhr.response);
  });
  
  this.$.myApi.setHeaders({});
  this.$.myApi.postSomething({
    aKey: "aValue"
  },
  function(res) {
    console.log(res);
  });
}
```


That's it! =)
