### JavaScript ES7 Function Bind Syntax

正是因为babel，我们可以使用最先进的javascript语法特性来进行开发，这是非常激动人心的。JavaScript ES7 Function Bind Syntax就是ES最新的语法。这里mark一下，以后看到别人的代码可以认识。

[参考文章](http://blog.jeremyfairbank.com/javascript/javascript-es7-function-bind-syntax/)

The Syntax

    // Binding a function to a context
    let log = ::console.log;
    ES5: var log = console.log.bind(console);

    // Calling functions with a context
    let foo = {};
    function bar() {
      log(this);
    }
    function world(a) {
      log(this, a);
    }
    foo::bar();
    function hello() {
      foo::world(...arguments);
    }
    
    ES5: 
    var foo = {};
    function bar() {
      log(this);
    }
    function world(a) {
      log(this, a);
    }
    bar.call(foo);
    function hello() {
      world.apply(foo, arguments);
    }

Iteration

ES5:

    var ArrayProto = Array.prototype;
    var map = ArrayProto.map;
    var filter = ArrayProto.filter;
    var todoItems = document.querySelectorAll('ul.my-list > li');
     
    var completedItems = filter.call(todoItems, function(item) {
      return item.dataset.completed;
    });
     
    var titles = map.call(todoItems, function(item) {
      return item.textContent;
    });

with function bind syntax：

    let { map, filter } = Array.prototype;
    let todoItems = document.querySelectorAll('ul.my-list > li');
     
    let completedItems = todoItems::filter(item => item.dataset.completed);
    let titles = todoItems::map(item => item.textContent);

Callbacks

ES5:

    var eventLib = require('eventLib');
    var self = this;
     
    eventLib.on('foo', function() {
      self.gotFoo();
    });
     
    eventLib.on('bar', this.gotBar.bind(this));
    eventLib.on('log', console.log.bind(console));

ES7:

    import eventLib from 'eventLib';
     
    eventLib.on('foo', ::this.gotFoo);
    eventLib.on('bar', ::this.gotBar);
    eventLib.on('log', ::console.log);    

Chaining