### 解构赋值（Destructuring assignment）

语法：

[a, b] = [1, 2]
[a, b, ...rest] = [1, 2, 3, 4, 5] //rest : [3,4,5]
{a, b} = {a:1, b:2}
{a, b, ...rest} = {a:1, b:2, c:3, d:4} //ES7

右侧可以是所有具有Iterator接口的数据解构

#### 数组解构赋值

Multiple-value returns

    function f() {
      return [1, 2];
    }

Ignoring some returned values
    
    [,,] = f();

#### 对象解构赋值

    var {x = 3} = {x : 5}; // 5

对象解构避免已使用代码块

    ({x} = {x:1});

模块的引用的时候经常会用到解构对象：

    let { log, sin, cos } = Math;

即：

    var log = Math.log;
    var sin = Math.sin;
    var cos = Math.cos;

又如：

     const { Loader, main } = require('toolkit/loader');

#### 字符串的解构赋值
    
    const [a, b, c, d, e] = 'hello';
    a // "h"
    b // "e"
    c // "l"
    d // "l"
    e // "o"
    let {length : len} = 'hello';
    len // 5

#### 函数参数的解构赋值

函数参数的默认值：

ES5 version
    
    function drawES5Chart(options) {
      options = options === undefined ? {} : options;
      var size = options.size === undefined ? 'big' : options.size;
      var cords = options.cords === undefined ? { x: 0, y: 0 } : options.cords;
      var radius = options.radius === undefined ? 25 : options.radius;
      console.log(size, cords, radius);
      // now finally do some chart drawing
    }

    drawES5Chart({
      cords: { x: 18, y: 30 },
      radius: 30
    });

ES6 version

    function drawES6Chart({size = 'big', cords = { x: 0, y: 0 }, radius = 25} = {}) //如果对应的参数没有传入值，则设置为默认值，注意：这里是为解构赋值的变量设置默认值，而不是为函数的参数设置默认值，函数参数的默认值会被当做一个整体看待
    {
      console.log(size, cords, radius);
      // do some chart drawing
    }

    drawES6Chart({
      cords: { x: 18, y: 30 },
      radius: 30
    });

使用模式赋值可以省却很多具体的数据结构构件和解析的代码，使代码更简洁。