# Create module with closure
```js
(function() {
    // This function immediately runs and all variables within here are private
}());



var moudle = (function () {
  function privateFunc() {...};

  return {
    add: function (a, b) {
      return a + b;
    },
    print: function (...x) {
      x.forEach(el => console.log(el););
    }
  }
}());

var Module = (function($, w, undefined) {
    // ...
    // return {...};
}(jQuery, window));

// The Revealing Module Pattern
var Module = (function() {
    // All functions now have direct access to each other
    var privateFunc = function() {
        publicFunc1();
    };
    var publicFunc1 = function() {
        publicFunc2();
    };
    var publicFunc2 = function() {
        privateFunc();
    };
    // Return the object that is assigned to Module
    return {
        publicFunc1: publicFunc1,
        publicFunc2: publicFunc2
    };
}());

// Module Pattern for Extension
(function($) {
    $.pluginFunc = function() {
        ...
    }
}(jQuery));

```

reference: [JavaScript Closures and the Module Pattern](https://www.joezimjs.com/javascript/javascript-closures-and-the-module-pattern/)
