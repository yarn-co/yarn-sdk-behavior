# APIBehavior
APIBehavior makes it super easy to interface with APIs in Polymer!

install with `bower install yarn-api-behavior`

**Here's what you need to know:**

1 ) This behavior is intended to be used to create a Polymer element for each API you plan on interfacing with.
2 ) Create an element, name it \<my-api\>. You need to include yarn-endpoints-behavior in your element as well, (downloadable by bower), so your api element's behaviors array should look like this: 

```javascript
behaviors: [
  YarnBehaviors.EndpointsBehavior,
  YarnBehaviors.APIBehavior
],
```
The order in the behaviors array is important.

3 ) Now in your api element, you configure endpoints in an "endpoints" property like so:
```javascript
/*
  Each endpoint is required to have these properties:
  method,
  path
*/

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
      path: '/path/to/endpoint',
      headers: {Authorization: "etWAe2092--ReallyLongJSONWebToken--:JSHDGSJDF34523sjdhj6klaksf244LHGJETHLIJ23l"}
    }
  });
}

/*
  Optional parameters for each endpoint are:
  protocol,
  host,
  headers
*/

/*
  There are properties api-behavior gives you that lets
  you set defaults for the host or protocol options.
  These values will be used in an endpoint when none
  are explicitly configured.
  
  (http still acts as the default protocol even if you don't set this.defaultProtocol.)
  
  Example:
  ready: function() {
    ...
    this.defaultProtocol = 'http';
    this.defaultHost = 'localhost:3001';
  }
*/
```
---

4 ) There's one thing left to do in your \<my-api\> element, that's bring in an iron-ajax element for api-behavior to latch onto. Here's what your api element's template should look like:
```
<template>
  <iron-ajax id='ajax'></iron-ajax>
</template>
```



And the rest is magic, so let's enjoy it!

Here's how to make a call to an endpoint:
In another element, you'll want to import your \<my-api\> element and stick it in the template. Give it an id, say, "myApi".

Now make calls to your endpoints like this:

```javascript
attached: function() {
  this.$.myApi.getFakeData(function(response){
    console.log(response);
  });
}
```

5 ) Let's talk about this for a second: for each endpoint you configured, api-behavior gave your element a function with the same name as you gave your endpoint. The way you call them differs depending on how you configured your endpoint.

If your endpoint method was GET, then the function will have these arguments--
```javascript
myApiElement.myEndpointName(callback[,parametersObject]);
// callback required, parameters optional
```
If your endpoint method was POST, PUT, or DELETE, the function will have these arguments--
```javascript
myApiElement.myEndpointName(bodyObject,callback[,parametersObject]);
// body, callback required, parameters optional
```

The callback will receive a response object and api-behavior assumes you'll be receiving JSON from the server.

Another example:
```javascript
attached: function() {
  this.$.myApi.getRealData(function(response){
    console.log(response);
  }, {
    param1: "thisIsAValue",
    param2: "o3rjhg93jth894hjtg924jg3589hjgi8jgiu349jg9348hgj894ghfj928gh876g4f6823dg7523fxrwcx64wxfd26vghr3bj4t9kn50ml680ko807jk596jgu34hfy2gc72eghfu3nt49hj358ghu24bfu32gc612vfdt12fdg713hf248hvu34hv82jiw1jd8123hf"
  });
}
```

6 ) That's it! =)
