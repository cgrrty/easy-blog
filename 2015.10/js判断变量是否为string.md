### js判断变量是否为string

    function isString(foo){
        return typeof(foo) === "string" || foo instanceof String;
    }

因为字符串有字面量和对象之分

类似的有：boolean，number

function不一样，因为function没有字面量

    var f = function(){}
    var f2 = new Function();
    console.log(typeof(f));//function
    console.log(typeof(f2));//function
    console.log(f instanceof Function);//true
    console.log(f2 instanceof Function);//true

---

另附方法2：
    
    function classOf(value) {
        return Object.prototype.toString.call(value);
    }   
    console.log(classOf(new String("")));//[object String]
    console.loeg(classOf(" "));//[object String]
