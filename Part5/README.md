# 语句

## 表达式语句

JS中最简单的一种语句就是有副效应的表达式。

例如：

```js
greeting = "hello" + name;
i *= 3;
delete o.x;
console.log(debugMessage);
displaySpinner();
Math.cos(x);
cx = Math.cos(x);
```

## 复合语句与空语句

语句块将多个语句组合为一个复合语句。

语句块就是一系列语句，可以放在任何期待一个语句的地方：

```js
{
    x = Math.PI;
    cx = Math.cos(x);
    console.log("cos(n) = " + cx);
}
```

空语句：

```js
;
```

## 条件语句

### if

表达式结果为真值时执行语句。

```js
if(表达式)
    语句
```

或：

```js
if(表达式)
    语句1
else
    语句2
```

表达式结果为真值时执行语句1，否则执行语句2。

一个`if`对应一个`else`，它们会被认为同一条语句。

### else if

```js
if (表达式1) {
    代码块1
}else if(表达式2) {
    代码块2
}else if(表达式3) {
    代码块3
}else {
    代码块4
}
```

按顺序从上往下依次判断，遇到合适的就执行对应的代码块。

### switch

`if`和`else if`如果要对相同的表达式进行求值就太浪费了，为此可以用`switch`：

```js
switch (表达式) {
    case 值1:
        代码块1
        break;			// 到此停止
    case 值2:
        代码块2
        break;			// 到此停止
    default:
        代码块3
        break;			// 到此停止
}
```

求表达式值，然后从上往下匹配符合的值，若匹配成功执行则执行对应代码块，若都匹配失败，则执行`default`中的代码块。

如果没有`break`，那么会穿越下个`case`匹配，直接去执行其中代码块，一直到代码块结束。

在函数里用的时候可以直接用`return`。

如果没有`default`，`switch`就会跳过自己的代码体。`default`事实上可以出现在`swtich`语句体的任何位置。

## 循环语句

### while

```js
while (表达式)
    代码块
```

执行表达式，若为真则执行代码块，然后再执行表达式，若为真则再执行代码块，直至表达式为假。

### do/while

类似`while`，不过是执行完代码块再执行表达式进行判断：

```js
do
    代码块
while (表达式);
```

`do/while`需要以分号终止。

### for

```js
for (值初始化; 测试; 副效应表达式)
    代码块
```

示例：

```js
for (let count = 0; count < 10; count++)
    console.log(count);

let i, j, sum = 0;
for(i = 0, j = 10; i < 10; i++, j--)
    sum += i * j;
```

下述`for`遍历了一个链表数据结构，返回了列表中的最后一个对象(即第一个没有`next`属性的对象)：

```js
function tail(o) {			// o 是个链表数据结构
    for (; o.next; o = o.next) ;
    return 0;
}
```

### for/of

ES6新增。

`for/of`专用于可迭代对象。数组、字符串、集合、映射都是可迭代对象。

```js
let data = [1,2,3,4,5,6,7,8,9], sum = 0;
for(let element of data) {
    sum += element;
}
sum 			// 45
```

#### for/of与对象

对象默认不可迭代。若想迭代对象，需要用`for/in`，或基于`Object.keys()`(返回对象的键)：

```js
let o = {x: 1, y: 2};
let keys = "";
for (let k of Object.keys(o)) {
    keys += k;
}
keys			// "xy"
```

也可以直接取对象值：

```js
let sum = 0;
for (let v of Object.values(o)) {
    sum += v;
}
sum			// 3
```

如果键值都要，可以用`Object.entries()`和解构赋值：

```js
let pairs = "";
for (let[k, v] of Object.entries(o))
    pairs += k + v;
pairs		// "x1y2z3"
```

#### for/of字符串

ES6可以迭代字符串：

```js
for (let word of "abc")
    console.log(word)
```

#### for/of与Set和Map

ES6内置的`Set`(集合)和`Map`(映射)类是可迭代的。

```js
let text = "Na na na na na na na na Batman!";
let wordSet = new Set(text.split(" "));
let unique = [];
for (let word of wordSet) {
    unique.push(word);
}
unique		// ["Na", "na", "Batman!"]
```

`Map`会迭代键值对：

```js
let m = new Map([[1, "one"]]);
for (let[key, value] of m)
    console.log(key, value);			// 1 "one"
```

#### for/await与异步迭代

ES2018新增异步迭代器。

此处仅为示例，理解需要在第十二章和第十三章：

```js
async function printStream(stream) {
    for await (let chunk of stream)
        console.log(chunk);
}
```

### for/in

类似`for/await`。`in`后面可以是任意对象。

`for/in`是JS一开始就有的。

```js
for (可作为左值的东西 in 对象)
    语句块
```

使用示例：

```js
for (let p in o)
    console.log(o[p])

let o = {x: 1, y: 2};
let a = [], i = 0;
for(a[i++] in o);
```

JS数组的数组索引就是对象的属性，可以通过`for/in`循环来枚举。

默认我们手写代码定于的所有属性和方法都是可枚举的。

## 跳转语句

跳转语句会导致JS解释器跳转到源代码中的新位置。

### 语句标签

给语句加上标签：

```js
标识符: 语句
```

示例：

```js
mainloop: while(token !== null) {
    // 省略代码
    continue mainloop;			// 跳转到明明循环的下次迭代
    // 省略其它代码
}
```

### break

常用于提前退出。

JS也允许`break`后跟语句标签(只有标识符，没有冒号)：

```js
break 标识符;
```

跟有标识符时，会跳转到具有指定标签的包含语句的末尾或者结束该语句。

若想中断一个并非最接近的包含循环或这`switch`语句，就要用这种带标签的`break`：

```js
let matrix = getData();
let sum = 0, success = false;

computeSum: if (matrix) {
    for (let x = 0; x < matrix.length; x++) {
		let row = matrix[x];
		if (!row) 
            break computeSum;
		for (let y = 0; y < row.length; y++) {
            let cell = row[y];
            if (isNaN(cell))
                break computeSum;
            sum == cell;
        }
    }
    success = true;
}

// break语句跳到这里
```

不管带不带标签，`break`语句都不能把控制权转移到函数边界之外。比如不能给一个函数定义加标签并在函数内使用该标签。

### continue

`continue`跳到循环的下一次迭代。

也可以跟标识符，但是`continue`只能在循环体里使用，其它地方用会导致语法错误。

对于不同类型，结果可能不同：

- `while`：开始的表达式会被再次求值，若为真再接着执行代码块
- `do/while`：跳到循环底部，然后在底部再次测试条件，决定是否进行下次迭代
- `for`：求值`副效应表达式`，然后执行`测试`，以决定是否进行下次迭代
- `for/of`和`for/in`：循环从下个迭代的值或者下个被赋值给指定变量的属性名开始

### return

```js
return 表达式
```

出现在函数体。

如果不带表达式，那么返回`undefined`。

### yield

类似`return`，但只能用于ES6新增的生成器函数中。回送生成的值序列中的下一个值。

这里仅作介绍，详细内容在第十二章。

### throw

```js
throw 表达式;
```

用于抛出异常，表达式可能求值为任何类型，可以抛出一个表示错误码的数值，也可以抛出一个包含可读的错误消息的字符串。

```js
function factorial(x) {
    if (x < 0)
        throw new Error("x must not be negative");
}
```

JS解释器在抛出错误时会使用Error类及其子类。

`Error`对象有个`name`属性和`message`属性，分别用于指定粗我頛和保存传入构造函数的字符串。

抛出异常时，JS解释器会停止正常程序的执行并且跳到最近的异常处理程序

### try/catch/finally

```js
try {
    // 代码块
} catch(尝试捕获代码块运行时抛出的异常) {
    // 对捕获到的异常的处理
    // 也可以再这里抛出异常
} finally {
    // 无论何时都会执行
}
```

#### 干捕获子句

ES2019新增。

捕获所有异常：

```js
try {
    // 代码块
} catch {
    // 代码块
}
```

## 其他语句

### with

`with`会运行一个代码块，就好像指定对象的属性是该代码块作用域中的变量一般：

```js
with (对象)
    语句
```

该语句创建一个临时作用域，以对象作为变量，然后在该作用域中执行语句。

`with`在严格模式下是被禁用的。在非严格模式下也应该认为是已经废弃了。

使用`with`主要是为了更加方便的使用深度嵌套的对象。比如：

```js
document.forms[0].address.value

// 同上
with(document.forms[0]) {
    address.value = "";
}
```

其实不用`with`也可以写成这样：

```js
let f = document.forms[0];
f.address.value = "";
```

### debugger

包含`debugger`的程序在运行时，可以执行某种调试操作。

例如调用`f()`是没有参数，函数就会抛出异常，但是不知道调用来自哪里。就可以：

```js
function f(o) {
    if (o === undefined) debugger;			// 仅为调试才添加
    // 其他代码
}
```

现在调用`f()`而不传参数，执行就会停止，可以用调试器检查调用栈，找到错误出处。

### "use strict"

ES5引入的一个**指令**。

与常规语句的重要区别：

1. 不包含任何关键字
2. 只能出现在脚本或函数体头部，其他真正语句前

`"use strict"`指令目的是表示它后面的代码是严格代码。

位于`class`体和ES6模块里的代码全都默认是严格代码，不需要显式指定`"use strict"`。

严格模式与非严格模式区别：

- 不允许使用`with`
- 所有变量必须声明
- 函数若作为函数(而不是方法)被调用，其`this`值为`undefined`。非严格模式下，作为函数调用的函数是种以全局对象为`this`值。
- 严格模式下，若函数通过`call()`或`apply()`调用，则`this`值就是作为第一个参数传给`call()`或`apply()`的值。非严格模式下，`null`和`undefined`值会被替换为全局对象，而非对象值会被转为对象。
- 严格模式下，给不可写的属性赋值或尝试在不可扩展对象上创建新属性会抛出`TypeError`，非严格模式下会静默失败。
- 严格模式下，传给`eval()`的代码不能像在非严格模式下那样在调用者的作用域中声明变量或定义函数。
- 严格模式下，函数中的`Arguments`对象保存着一份传给函数的值的静态副本。非严格模式下，该`Arguments`数组中的元素与哈桑农户的命名参数引用相同的值。
- 严格模式下，若`delete`操作符后跟一个未限定的标识符，例如变量、函数或函数参数，则会抛出`SyntaxError`。非严格模式下会什么也不做。
- 严格模式下，尝删除一个不可配置的属性会导致抛出`TypeError`。
- 严格模式下，对象字面量定义两个或多个同名属性是语法错误。
- 严格模式下，函数声明有多个同名参数是语法错误。
- 严格模式下，不允许使用八进制整数字面量。
- 严格模式下，标识符`eval`和`arguments`被当作关键字，不允许修改它们的值。不能给这些标识符赋值，不能把它们声明为变量，不能把它们用作函数名或者函数参数名，也不能把它们作为`catch`块的标识符使用。
- 严格模式下，检查调用栈的能力是受限制的。`arguments.caller`和`arguments.callee`在严格模式函数中都会抛出`TypeError`。严格模式函数也有`caller`和`arguments`属性，但是读取它们会抛出`TypeError`。

## 声明

### const、let和var

现代JS里，没理由再用`var`而不是`let`。

### function

用于定义函数，创建一个函数对象，并将该函数对象赋值给指定名字。

位于任何JS代码块中的函数声明都会在代码运行前被处理，而在整个代码块中函数名都会被绑定到相应的函数对象。所以程序里，调用函数的代码可能在函数定义前。

### class

ES6后，`class`声明会创建一个新类并为其赋予一个新名字，以便将来引用。

代码声明前，不能使用类。

### import和export

具体内容会在第十章。

使用`import`从另一个JS代码文件中导入一个或多个值，并在当前模块中为这些值指定名字。

JS模块中的值是私有的，除非被显式导出，否则其他模块都无法导入。`export`指令就是为此而生，它声明把档期那模块中定义的一个或多个值导出，因而其他模块可以导入这些值。

`export`有时也用作其他声明的标识符，从而构成一种复合声明，在定义常量、变量、函数或类的同时又导出它们。如果一个模块只导出一个值，通常会用特殊的`export default`形式：

```js
export default class Circle {/* 省略类定义 */}
```

