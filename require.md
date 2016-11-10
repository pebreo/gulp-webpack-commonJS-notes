OVERVIEW
--------
`require` is a powerful feature in CommonJS (a node standard).
With `require` we can import code easily from our installed
`node_modules` or from other parts of the code base through
file paths.

For example, if I wanted to use a function from a
different file, I could do this:

```javascript
// file1.js
// module.exports is an object that node uses to 'export' to other files

module.exports = function () {
  console.log("hello world")
}

// file2
var func = require('./file1.js');
func();
```
// file1.js
Another example:
```javascript
var exports = {
    myfunc: function() { console.log('hello'); }
    myvar: 3;
}
module.exports = exports;

// file2.js
var obj = require('./file1.js');
obj.myfunc();
console.log(obj.myvar);
```


Reference
https://docs.nodejitsu.com/articles/getting-started/what-is-require/