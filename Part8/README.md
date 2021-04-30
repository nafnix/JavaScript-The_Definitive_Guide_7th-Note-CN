# 函数

子例程(subroutine)或过程(procedure)就是函数。

除了参数，每个调用还有一个值，即调用上下文(invocation context)：`this`。

设计用来初始化一个新对象地函数被称为构造函数(constructor)。

JS中函数也是对象。

JS函数是闭包(closure)，基于闭包可以实现重要且强大的编程技巧。

## 定义函数

### 函数声明

函数声明定义的函数会被提升到顶部，这样在定义函数语句前就可以调用它们。

```js
function 函数名(形参) {
    执行代码块
    return 返回值
}
```

没有返回值时，调用函数返回`undefined`。

### 函数表达式

函数表达式在它们的变量被求值前是不存在的。

函数名对定义为表达式的函数而言是可选的：

```js
const square = function(x) { return x*x; };

// 函数表达式可以包含名字 这对递归有用
const f = function fact(x) { if (x <= 1) return 1; else return x*fact(x-1); };

// 函数表达式也可以用作其他函数的参数
[3,2,1].sort(function(a, b) { return a-b; });

// 函数表达式也可以定义完立即调用
let tensquared = (functoin(x) { return x*x; }(10));
```

### 箭头函数

ES6后新增：

```js
const sum = (x, y) => {return x + y;};
```

因为箭头函数是表达式而不是语句，所以不必使用`function`关键字，而且也不需要函数名。

若函数体只有一个语句，那也可以省略`return`关键字：

```js
const sum = (x, y) => x + y;
```

若只有一个参数：

```js
cosnt polynomial = x => x*x + 2*x + 3;
```

没有参数的话，必须要把圆括号写出来：

```js
const constantFunc = () => 42
```

箭头函数的简洁欸语法让它们非常适合作为值传给其他函数，这在使用`map()`、`filter()`和`reduce()`等数组方法是很常见的：

```js
// 得到一个过滤掉null元素的数组
let filtered = [1, null, 2, 3].filter(x => x !== null);		// filtered == [1, 2, 3]

// 求数值的平方
let squares = [1,2,3,4].map(x => x*x);
```

箭头函数没有`prototype`属性，意味箭头函数不能作为新类的构造函数。

### 嵌套函数

```js
function hypotenuse(a, b) {
    function square(x) { return x*x; }
    return Math.sqrt(square(a) + square(b));
}
```

## 调用函数

### 函数调用

```js
printprops({x: 1});
let total = distance(0,0,2,1) + distance(2,1,3,5);
let probability = factorial(5)/factorial(13);
```

先求值实参表达式，结果作为函数的实参。

#### 条件式调用

ES2020中：

```js
(f !== null && f !== undefined) ? f(x) : undefined
```

对于非严格模式下的函数调用，调用上下文(`this`)是全局对象。但是在严格模式下，调用上下文是`undefined`。但使用箭头语法定义的函数不同：它们总是继承自身定义所在环境的`this`值。

要作为函数(而非方法)来调用的函数通常不会在定义中使用`this`，但可以在这些函数中使用`this`来确定是否处在严格模式：

```js
// 定义并调用函数 以确定当前是不是严格模式
const strict = (function() { return !this; }());
```

如果函数调用自己达到上万次，可能会导致"最大调用栈溢出"(Maximum call-stack size exceeded)错误。

###  方法调用

```js
o.m = f;

o.m();

o.m(x, y);

let calculator = {
    operand1: 1,
    operand2: 2,
    add() {
        this.result = this.operand1 + this.operand2;
    }
};

calculator.add()			// 1+1
calculator.result			// 2
```

其他调用方法：

```js
o["m"](x, y)
a[0](z)

customer.surname.toUpperCase();		// 调用customer.surname的方法
f().m()								// 在f()的返回值上调用m()
```

若方法返回对象，那么基于该方法调用的返回值还可以继续调用其他方法。这样就能得到表现为一系列方法调用(或方法调用链)。

除了箭头函数，嵌套函数不会继承包含函数的`this`值。若嵌套函数被当作方法来调用，那么它的`this`值就是调用它的对象。

常见错误：对定义在方法中的嵌套函数，以为可以用`this`获得该方法的调用上下文。

```js
let o = {					// 对象o
    m: function() {			// 对象方法 m
        let self = this;	// 将this存于变量中
        this === 0;			// true this是对象o
        f();				// 调用嵌套函数f()
        
        function f() {		// 嵌套函数f
            this === o;		// false this是全局对象或undefined
            self === o;		// true
        }
    }
}
```

ES6后，解决该问题的另一个技巧是把嵌套函数`f`转为箭头函数，因为箭头函数可以继承`this`值：

```js
const f = () => {
    this === o;			// true 因为箭头函数继承this
}
```

需要把该函数`f`的定义放到调用`f`函数的代码前。

也可以用嵌套函数的`bind()`方法，以定义一个在指定对象上被隐式调用的新函数：

```js
const f = (function() {
    this === o;			// true 将该函数绑定到了外部的this
}).bind(this);
```

### 构造函数调用

如果没有参数列表，那么可以省略括号。

```js
o = new Object();
o = new Object;
```

构造函数会创建一个新的空对象，该对象继承构造函数的`prototype`属性指定的对象。

构造函数正常情况下不使用`return`关键字，而是初始化新对象并在到达函数体末尾时隐式返回该对象。如果构造函数显式使用了`return`语句返回某个对象，那么该对象会变成调用表达式的值。

### 间接调用

`call()`和`apply()`可以用来间接调用函数。这两个方法允许我们指定调用时的`this`值，意味着我们可以将任意函数作为任意对象的方法来调用，即使该函数实际上并不是该对象的方法。

### 隐式函数调用

有些JS语言特性看着不像函数调用，但实际上会导致某些函数被第哦啊用。

下述是可能导致隐式函数调用的一些语言特性：

- 若对象有获取方法或设置方法，则查询或设置其属性值可能会调用这些方法
- 当对象在字符串上下文中使用时(例如拼接对象和字符串时)，会调用对象的`toString()`方法
- 在遍历可迭代对象的元素时，也会设计一系列方法调用
- 标签模板字面量是一种伪装的函数调用
- 代理对象的行为完全由函数控制。这些对象上的几乎任何操作都会导致一个函数被调用

## 函数实参与形参

JS函数定义不会指定函数形参的类型，函数调用也不对传入的实参进行任何类型检查，也不对传入实参的个数做检查。

### 可选形参与默认值

当调用函数传入的实参少于声明的形参时，额外的形参会获得默认值，通常是`undefined`。

可选参数一般放在参数列表最后，这样在调用时才可以省略。

ES6后，可以给参数定义默认值：

```js
function getPropertyNames(o, a = []) {
    for(let property in o) a.push(property);
    return a;
}
```

函数的形参默认值表达式会在函数调用时求值，不会在定义时求值。

如果函数有多个形参，则可以使用前面参数的值来定义后面参数的默认值：

```js
const rectangle = (width, height=width*2) => ({width, height});
rectangle(1)			// {width: 1, height: 2}
```

### 剩余型参与可变长度实参列表

剩余形参(rest parameter)的作用：让我们能够编写在调用时传入比形参多任意数量的实参的函数。

```js
function max(first=-Infinity, ...rest) {
    let maxValue = first;
    for (let n of rest) {
        if (n > maxValue){
            maxValue = n;
        }
    }
    return maxValue;
}

max(1, 10, 100, 2, 3, 1000, 4, 5, 6)		// 1000
```

可以接受任意数量实参的函数称为可变参数函数(variadic function)、可变参数数量函数(variable arity function)或变成函数(vararg function)。本书使用最通俗的"变长函数"(vararg)，该称呼可以追溯到C编程语言诞生的时期。

### Argument对象

ES6前，变长函数基于`Argument`对象实现。

`Argument`对象是一个类数组对象，允许通过数值而非名字取得传给函数的参数值。

重写`max`：

```js
function max(x) {
    let maxValue = -Infinity;
    for(let i = 0; i < arguments.length; i++) {
        if (arguments[i] > maxValue) maxValue = arguments[i];
    }
    return maxValue;
}

max(1, 10, 100, 2, 3, 1000, 4, 5, 6)		// 1000
```

`Argument`的效率低且难优化，所以应该尽量避免使用它。

严格模式下，不能使用`arguments`作为函数形参或局部变量。

### 在函数调用中使用扩展操作符

```js
let numbers = [5, 2, 10, -1, 9, 100, 1];
Math.min(...numbers)		// -1
```

如下函数所示，接受一个函数实参并返回该函数的可测量版本，以用于测试：

```js
// 该函数接受一个函数并返回一个包装后的版本
function timed(f) {
    return function(...args) {	// 把实参收集到一个剩余形参数组args中
        console.log(`Entering function $(f.name)`);
        let startTime = Date.now();
        try {
            return f(...args);
        }
        finally {
            console.log(`Exiting $(f.name) after ${Date.now()-startTime}ms`)
        }
    };
}

function benchmark(n) {
    let sum = 0;
    for(let i = 1; i <= n; i++) sum += i;
    return sum;
}

timed(benchmark)(1000000)		// 500000500000
```

### 把函数实参解构为形参

```js
function vectorAdd([x1, y1], [x2, y2]) {
    return [x1 + x2, y1 + y2];
}
vectorAdd([1, 2], [3, 4])		// [4, 6]

function vectorMultiply({x, y}, scalar) {
    return {x : x*scalar, y: y*scalar};
}
vectorMultipy({x: 1, y: 2}, 2)		// {x: 2, y: 4}
```

如果把解构的属性赋给不同的名字，那代码会更长也更不好理解。

解构赋值也可以定义形参默认值：

```js
function vectorMultiply({x, y, z=0}, scalar) {
    return {x: x*scalar, y: y*scalar, z: z*scalar};
}
vectorMultiply({x: 1, y: 2}, 2)		// {x: 2, y: 4, z: 0}
```

JS不支持指定实参名传值调用，但是可以通过把对象参数结构为函数参数来模拟。

例如有个函数从指定数组中把指定个数的元素复制到另一个数组中，参数中可选地指定每个数组的起始索引值。此时至少要涉及5个参数，其中有的有默认值，而调用者又很难记住这些参数的顺序，为此可以像下面这样定义`arraycopy()`函数：

```js
function arraycopy({from, to=from, n=from.length, fromIndex=0, toIndex=0}) {
    let valuesToCopy = from.slice(fromIndex, fromIndex + n);		// 0-4
    to.splice(toIndex, 0, ...valuesToCopy);
    return to;
}

let a = [1,2,3,4,5], b = [9,8,7,6,5];
arraycopy({from: a, n: 3, to: b, toIndex: 4})		// [9,8,7,6,1,2,3,5]

/*
arraycopy({
	from=[1,2,3,4,5],
	to=[9,8,7,6,5],
	n=3,
	fromIndex=0,
	toIndex=4;
})
*/
```

解构数组的时候，可以给被展开数组中的额外元素定义一个剩余形参：

```js
function f([x, y, ...coords], ...rest) {
    return [x+y, ...rest, ...coords];
}
f([1, 2, 3, 4], 5, 6)			// [3, 5, 6, 3, 4]
```

ES2018中，解构对象时也可以使用剩余形参。此时剩余形参的值是一个对象，包含所有为被解构的属性。对象剩余形参经常和对象扩展操作一起用：

```js
function vectorMultiply({x, y, z=0, ...props}, scalar) {
    return {x: x*scalar, y: y*scalar, z: z*scalar, ...props};
}
vectorMultiply({x: 1, y: 2, w: -1}, 2)		// {x: 2, y: 4, z: 0, w: -1}
```

### 参数类型

可以用描述性强的名字作为函数参数，同时通过在注释里解释函数的参数来解决该问题。

## 函数作为值

```js
let a = [x => x*x, 20];
a[0](a[1])				// 400
```

函数作为值示例：

```js
const operators =  {
    add: 		(x, y) => x+y,
    subtract:	(x, y) => x-y,
    multiply:	(x, y) => x*y,
    divide:		(x, y) => x/y,
    pow:		Math.pow		// 预定义的函数也没问题
};

function operate2(operation, operand1, operand2) {
    if (typeof operators[operation] === "function") {
        return operators[operation](operand1, operand2);
    }
    else throw "unknow opeartor";
}
operate2("add", "hello", operate2("add", " ", "world"))		// "hello world"
operate2("pow", 10, 2)			// 100
```

### 定义自己的函数属性

如果函数需要一个"静态"遍历，该变量的值需要在函数每次调用时返回一个唯一的数。

```js
uniqueInteger.counter = 0;

function uniqueInteger() {
    return uniqueInteger.counter++;
}

uniqueInteger()		// 0
uniqueInteger()		// 1
```

下述的`factorial`函数使用了自身的属性来缓存之前计算的结果：

```js
function factorial(n) {
    if(Number.isInteger(n) && n > 0) {
        if(!(n in factorial)) {
            factorial[n] = n * factorial(n-1);
        }
        return factorial[n];
    } else {
        return NaN;
    }
}

factorial[1] = 1;		// 初始化缓存
factorial(6)			// 720
factorial[5]			// 120
```

## 函数作为命名空间

可以在一个表达式中定义并调用匿名函数：

```js
(function() {
    // 复用代码
}());			// 函数定义结束后立即调用它 
```

在一个表达式中定义并调用匿名函数的技术非常常用，因此有了别称："立即调用函数表达式"(immediately invokeyd function expression)。

## 闭包

JS也使用词法作用域(lexical scoping)：函数执行时使用的是定义函数时生效的变量作用域，而非调用函数时生效的变量作用域。

为实现词法作用域，JS函数对象的内部状态不仅要包括函数代码，还要包括对函数定义所在作用域的引用。这种函数对象与作用域组合起来解析函数函数变量的机制，称为闭包(closure)。

闭包本质：捕获自身定义所在外部函数的局部变量及参数绑定。

闭包可以捕获一次函数调用的局部变量，可以将这些变量作为私有状态：

```js
let uniqueInteger = (function() {		// 定义并调用
    let counter = 0;					// 下面函数的私有状态
    return function() {return counter++;};
}())
uniqueInteger()			// 0
uniqueInteger()			// 1
```

第一行代码定义并调用了一个函数，因此真正赋给`uniqueInteger`的是该函数的返回值，而该函数的返回值是嵌套在该函数里的一个函数。而内嵌函数有权访问其作用域里的变量，而且可以用定义在外部函数里的变量`counter`。外部函数一旦返回，就没有别的代码能够看到变量`counter`了，此时内部函数拥有对它的专有访问权。

同一个外部函数可以定义两个或者更多嵌套函数：

```js
function counter() {
    let n = 0;
    return {
        count: function() { return n++; },
        reset: function() { n = 0; }
    };
}
let c = counter(), d = counter();
c.count()
d.count()
```

调用两次`counter`可以得到两个不同私有变量的计数器对象。

可以将这种闭包技术与属性获取方法和设置方法组合使用：

```js
function counter(n) {
    return {
        get count() { return n++; }
        
        // 不允许n值减少
        set count(m) {
            if (m > n) n = m;
            else throw Error("count can only be set to a larger value");
        }
    };
}
let c = counter(1000);
c.count			// 1000
c.count			// 1001
c.count = 2000
c.count			// 2000
c.count = 2000	// "count can only be set to a larger value"
```

只使用参数`n`保存供属性访问器方法共享的私有状态。这样可以让`counter`的调用者指定私有变量的初始值。

什么情况下闭包会意外地共享访问不该被共享的变量：

```js
function constfunc(v) { return () => v; }
let funcs = [];
for(var i = 0; i < 10; i++) 
    funcs[i] = constfunc(i);

funcs[5]()		// 5
```

在编写这种使用循环创建多个闭包的代码时，一个常见的错误是把循环转移到定义闭包的函数里：

```js
function constfuncs() {
    let funcs = [];
    for(var i = 0; i < 10; i++) {
        funcs[i] = () => i;
    }
    return funcs;
}

let funcs = constfuncs();
funcs[5]()				// 10 当constfuncs返回后变量i的值是10 全部10个闭包共享该值 funcs[0-9]()都是10
```

关键：与闭包关联的作用域是"活的"。嵌套函数不会创建作用域的私有副本或截取变量绑定的静态快照。

代码中`for`循环的`var i`声明循环遍历，所以变量`i`的作用域是整个函数体，而非仅仅只是循环块，所以只要把`var`替换成`let`或`const`就行了。

闭包注意事项：`this`是JS关键字，而非变量。箭头函数继承包含它们的函数中的`this`值，但是使用`function`定义的函数并非如此。

## 函数属性、方法与构造函数

### length属性

表示函数的元数(arity)，即函数在参数列表中生命的形参个数。函数的剩余形参不包含在`length`属性其中。

### name属性

表示定义函数时使用的名字。

若是未命名函数，则表示在第一次创建该函数时赋给该函数的变量名或属性名。

### prototype属性

除了箭头函数，所有函数都有`prototype`属性，该属性引用一个被称为原型对象的对象。

当函数被作为构造函数使用时，新创建的对象从原型对象继承属性。

### call()和apply()方法

这两个方法允许间接调用一个函数。

`call()`和`apply()`的第一个参数都是要在其上调用该函数的对象，也就是函数的调用上下文，在函数体内会变成`this`关键字的值。

要把函数`f`作为对象`o`的方法进行调用(不传参数)，可以用它们：

```js
f.call(o)
f.apply(o)

// 类似如下代码 假设o没有属性m 临时创建一个方法
o.m = f;
o.m();
delete o.m;
```

其他的参数会传给被调用的函数：

```js
f.call(o, 1, 2);		// 将函数f作为o对象的方法使用 并给f传入参数1, 2
```

`apply`要以数组形式传入：

```js
f.apply(o, [1, 2]);
```

ES5：在不适用扩展操作符的情况下，找到一个数值数组中的最大值，可以用`apply`把数组中的元素传给`Math.max`：

```js
let biggest = Math.max.apply(Math, arrayOfNumbers);
```

定义一个类似前面写过的`timed`的函数：

```js
function trace(o, m) {
    let original = o[m];						// 在闭包中记住原始方法
    o[m] = function(...args) {					// 定义新方法
        console.log(new Date(), "Entering:", m);		// 打印消息
        let result = original.apply(this, args);		// 调用原始方法
        console.log(new Date(), "Exiting:", m);			// 打印消息
        return result;									// 返回结果
    };
}
```

### bind()方法

该方法主要是将函数绑定到对象。

若在函数`f`上调用`bind()`并传入对象`o`，则该方法返回一个新函数。若作为函数来调用该新函数，就会像`f`是`o`的方法一样调用原始函数。传给该新函数的所有参数都会传给原始函数。

```js
function f(y) {return this.x + y;}		// 该函数需要绑定
let o = {x: 1};
let g = f.bind(o);
g(2)			// 3
let p = {x: 10, g};
p.g(2)			// 3
```

箭头函数从定义它们的环境里继承`this`值，该值不能被`bind()`覆盖。

`bind()`还可以做其他事，例如执行"部分应用"，即在第一个参数之后传给`bind()`的参数也会随着`this`值一起被绑定。

部分应用是函数式变成中的一个常用技巧，有时候被称为柯里化(currying)。示例：

```js
let sum = (x, y) => x + y;			// 返回2个参数之和
let succ = sum.bind(null, 1);		// 把第一个参数(x)绑定为1
succ(2)				// 3 x绑定到1 2会传给参数y

function f(y, z) { return this.x + y + z; }
let g = f.bind({x: 1}, 2);			// 绑定this和y
g(3)			// 6 this.x=1 y=2 z=3
```

### toString()方法

ECMAScript要求该方法返回一个符合函数声明语句的字符串。

### Function()构造函数

```js
const f = new Function("x", "y", "return x*y;");
// 等同于
const f = function(x, y){ return x*y; };
```

`Function`最后一个参数是函数体的文本。可以包含任意JS语句，相互以分号分隔。其他前面的字符串都用于指定新函数的参数名。

`Funtion`创建的也是匿名函数。

- `Function`允许在运行时动态创建和编译JS函数
- `Function`构造函数每次被调用时都会解析函数体并创建一个新的函数对象。
  - 若在循环中或频繁调用的函数中出现了对它的调用，可能影响程序性能。
  - 相对而言，出现在循环的嵌套函数和函数表达式不会每次都被重新编译。
- 它创建的函数不使用词法作用域，而是始终编译为如同顶级函数一般。

```js
let scope = "global";
function constructFunction() {
    let scope = "local";
    return new Function("return scope");		// 不会捕获局部作用域
}

constructFunction()()		// "global"
```

最好是把`Function`构造函数作为在自己私有作用域中定义新变量和函数的`eval()`的全局作用域版。

## 函数式编程

### 使用函数处理数组

略。

### 高阶函数

即操作函数的函数，接收一个或多个函数作为参数并返回一个新函数。

示例：

```js
function mapper(f) {
    return a => map(a, f);
}

const increment = x => x+1;
const incrementAll = mapper(increment);
incrementAll([1, 2, 3])		// [2, 3, 4]
```

更通用的高阶函数示例：

```js
function compose(f, g) {
    return function(...args) {
        return f.call(this, g.apply(this, args));
    };
}

const sum = (x, y) => x+y;
const square = x => x*x;
compose(square, sum)(2, 3)		// 25

/*
tmp = square(sum(x, y))
sum(x, y): 5
square(5): 25
tmp = 25
*/
```

### 函数的部分应用

函数的`bind()`方法返回一个新函数，这个新函数在指定的上下文中以指定的参数调用`f`。

我们说该函数绑定到了一个对象并部分应用了参数，`bind()`方法在左侧部分应用参数，也就是传给`bind()`的参数会放在传给原始函数的参数列表开头。但也有可能在右侧部分应用参数：

```js
function partialRight(f, ...outerArgs) {
    return function(...innerArgs) {						// 返回该函数
        let args = [...innerArgs, ...outerArgs];		// 构建参数列表
        return f.apply(this, args);						// 通过它调用f
    }
}

// 将内部参数表合并到外部参数表中 该参数列表中的undefined值会被来自内部参数列表的值填充
function partial(f, ...outerArgs) {
    return function(...innerArgs) {
        let args = [...outerArgs];			// 外部参数模板的局部副本
        let innerIndex = 0;						// 内部参数索引
        
        // 循环遍历args 用内部参数填充undefined值
        for(let i = 0; i < args.length; i++) {
            if(args[i] === undefined) 
                args[i] = innerArgs[innerIndex++];
        }
        
        // 现在把剩余的内部参数都加进来
        args.push(...innerArgs.slice(innerIndex));
        return f.apply(this, args);
    };
}
```

### 函数记忆

在定义自己的函数属性中，我们定义了一个缓存自己之前计算结果的阶乘函数。函数式编程里，这种缓存被称为函数记忆(memoization)。

下述代码展示了高阶函数`memoize()`可以接收一个函数参数，然后返回该函数的记忆版：

```js
// 返回f的记忆版
// 只适用于f的参数都有完全不同的字符串表示的情况
function memozie(f) {
    const cache = new Map();		// cache保存在该闭包中
    
    return function(...args) {
        // 创建参数的字符串版本 以用作缓存键
        let key = args.length + args.join("+");
        if (cache.has(key)) {
            return cache.get(key);
        } else {
            let result = f.apply(this, args);
            cache.set(key, result);
            return result;
        }
    };
}
```

- 该`memoize`创建了一个新对象作为缓存使用，并将该对象赋给一个局部变量，从而让它成为被返回的函数的私有变量。
- 返回的函数将其参数数组转为字符串，并使用该字符串作为缓存对象的属性。
- 如果缓存存在某个值，将直接返回该值；否则调用指定的函数计算这些参数的值，然后缓存该值，最后返回该值。

