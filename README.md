# APIBehavior
APIBehavior makes it super easy to interface with APIs in Polymer!

install with `bower install yarn-api-behavior`

**Here's what you need to know:**

1 ) This behavior is intended to be used to create a Polymer element for each API you plan on interfacing with.

2 ) Create an element, name it \<my-api\>.

3 ) In your api element, you configure endpoints in an "endpoints" property like so:

## Configuring Endpoints
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
      protocol: 'http',
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


## Making calls to endpoints

For each endpoint configured, api-behavior generates a function on your api element with the same name you gave the respective endpoint.

In another element, you'll want to import your \<my-api\> element and stick it in the template. Give it an id, say, "myApi".

Now make calls to your endpoints like this:

```javascript
attached: function() {
  this.$.myApi.getFakeData(function(response){
    console.log(response);
  });
}
```

If your endpoint method was GET, then the function will have these arguments--
```javascript
myApiElement.myEndpointName(callback[,parametersObject]);
// 'callback' is required, 'parameters' is optional
```
If your endpoint method was POST, PUT, or DELETE, the function will have these arguments--
```javascript
myApiElement.myEndpointName(body,callback[,parametersObject]);
// 'body', 'callback' are required, 'parameters' is optional
```

The callback will receive a response object and api-behavior assumes you'll be receiving JSON from the server.
If you want the response to be a different type, then put the response type in a 'handleAs' property for that
endpoint.

More examples:
```javascript
attached: function() {
  this.$.myApi.postSomething(
  {
    aKey: "aValue"
  },
  function(res) {
    console.log(res);
  });
  
  this.$.myApi.getRealData(function(res){
    console.log(res);
  }, {
    param1: "aValue1",
    param2: "aValue2"
  });
}
```


That's it! =)
