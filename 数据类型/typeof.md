## typeof的核心源码

```
JS_PUBLIC_API(JSType)
    JS_TypeOfValue(JSContext *cx, jsval v)
    {
        JSType type = JSTYPE_VOID;
        JSObject *obj;
        JSObjectOps *ops;
        JSClass *clasp;

        CHECK_REQUEST(cx);
        if (JSVAL_IS_VOID(v)) {  // (1)
            type = JSTYPE_VOID;
        } else if (JSVAL_IS_OBJECT(v)) {  // (2)
            obj = JSVAL_TO_OBJECT(v);
            if (obj &&
                (ops = obj->map->ops,
                 ops == &js_ObjectOps
                 ? (clasp = OBJ_GET_CLASS(cx, obj),
                    clasp->call || clasp == &js_FunctionClass) // (3,4)
                 : ops->call != 0)) {  // (3)
                type = JSTYPE_FUNCTION;
            } else {
                type = JSTYPE_OBJECT;
            }
        } else if (JSVAL_IS_NUMBER(v)) {
            type = JSTYPE_NUMBER;
        } else if (JSVAL_IS_STRING(v)) {
            type = JSTYPE_STRING;
        } else if (JSVAL_IS_BOOLEAN(v)) {
            type = JSTYPE_BOOLEAN;
        }
        return type;
    }
```
上面JS引擎的代码看起来只是一些逻辑判断，而底层api（JSVAL_IS_*）的判断是根据数据的低位标识来判断的，有一个经典的bug需要知道的。
```
console.log(typeof null);
// output: "object"
```
这是js引擎的bug，并且一直没法修复，说是修复了会影响到其他核心代码，所以一直遗留到现在。出现这个情况到原因是 null 和 object的低位标识都是000。除了上面的问题我们还要避免使用以下的代码。
### 注意避免使用以下例子

```
typeof new Boolean(true) === 'object';
typeof new Number(1) === 'object';
typeof new String('abc') === 'object';
```
## 总结

typeof只适用于判断基本的数据类型，对于引用类型的判断需要用instanceof
```
// 数值
typeof 37 === 'number';
typeof Infinity === 'number';
typeof NaN === 'number'; 
typeof 42n === 'bigint';

// 字符串
typeof 'bla' === 'string';

// 布尔值
typeof true === 'boolean';

// Symbols
typeof Symbol('foo') === 'symbol';
typeof Symbol.iterator === 'symbol';

// Undefined
typeof undefined === 'undefined';
typeof declaredButUndefinedVariable === 'undefined';
typeof undeclaredVariable === 'undefined'; 

// 对象
typeof {a: 1} === 'object';

// 使用 Array.isArray 或者 Object.prototype.toString.call
// 区分数组和普通对象
typeof [1, 2, 4] === 'object';

typeof new Date() === 'object';
typeof /regex/ === 'object'; // 历史结果请参阅正则表达式部分


// 下面的例子令人迷惑，非常危险，没有用处。避免使用它们。
typeof new Boolean(true) === 'object';
typeof new Number(1) === 'object';
typeof new String('abc') === 'object';

// 函数
typeof function() {} === 'function';
typeof class C {} === 'function'
typeof Math.sin === 'function';
```

### 参考资料
[MDN typeof 解析](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/typeof)

[为什么typeof === 'object'](https://2ality.com/2013/10/typeof-null.html)