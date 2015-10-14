undefined不是保留字

undefined不是字面量

undefined是Undefined类型的唯一值

undefined是标识符

undefined在Esprima的解释结果：

{
    "type":"ExpressionStatement",
    "expression":{
        "type":"Identifier",
        "name":"undefined"
     }
}

以下代码在node中的运行结果：
    
    var undefined = 1;
    console.log(undefined);

结果为1
