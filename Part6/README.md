# 对象

## 对象简介

对象是个属性的无序集合，每个属性都有名字和值。

JS对象可以从其他对象继承属性，这个其他对象称为其"原型"。

对象的方法通常是继承来的属性，这种"原型式继承"也是JS的主要特性。

JS中，任何不是字符串、数值、符号、布尔值、`null`、`undefined`的值都是对象。

即使字符串、数值、布尔值都不是对象，它们的的行为也类似不可修改的对象。

```js
let y = x;			// y是变量x的一个引用 而非副本
```

JS使用术语"自由属性"指代非继承属性。

除名字和值之外，每个属性还有3个属性特性：

1. 可写：指定是否可以设置属性的值
2. 可枚举：指定是否可以在`for/in`循环中返回属性的名字
3. 可配置：指定是否可以删除属性，以及是否可以修改其特性

## 创建对象

### 对象字面量

```js
let empty = {};						// 没有属性的对象
let point = {x: 0, y: 0};			// 包含两个数值属性
let p2 = {x: point.x, y: point.y};	// 值比较复杂
let book = {
    "main title": "JavaScript",			// 属性名包含空格和连字符 因此使用字符串字面量
    "sub-title": "The Definitive Guide",
    for: "all audiences",			// for是保留字 但是没有引号
    author: {						// 该属性的值是个对象
        firstname: "David",
        surname: "Flanagan"
    }
}
```

### 使用new创建对象

```js
let o = new Object();		// 创建一个空对象 与{}相同
let a = new Array();		// 创建一个空数组 与[]相同
let d = new Date();			// 创建一个表示当前时间的日期对象
let r = new Map();			// 创建一个映射对象 用于存储键/值映射
```

### 原型

几乎每个JS对象都有另一个与之关联的对象，这另一个对象称为原型(prototype)，第一个对象从这个原型继承属性。

通过对象字面量创建的所有对象都有相同的原型对象，可以通过`Object.prototype`引用该原型对象。

使用`new`和构造函数调用创建的对象，使用构造函数`prototype`属性的值作为它们的原型。使用`new Object()`创建的对象继承自`Object.prototype`，与通过`{}`创建的对象相同。

`Object.prototype`没有原型，因为它不继承任何属性。

### Object.create()

`Object.create()`用于创建一个新对象，接受一个参数作为新对象的原型：

```javascript
let o1 = Object.create({x: 1, y : 2});
```

创建一个没有原型的新对象，这种新对象不会继承任何东西，甚至`toString()`都没有：

```js
let o2 = Object.create(null);
```

创建一个普通空对象(`{}`或`new Object()`返回的对象)：

```js
let o3 = Object.create(object.prototype);
```

`Object.create()`的一个用途是防止对象被某个第三方库函数意外修改。这种情况下，不要直接把对象传给库函数，而应该传给一个继承自它的对象。如果函数读取这个对象的属性，可以读到继承的值。而如果它设置这个对象的属性，那么修改不会影响对原始对象。

```js
let o = {x: "don't change this value"};
library.function(Object.create(o));			// 防止意外修改
```

## 查询和设置属性

```js
let author = book.author;			// 取得book的"author"属性
let name = book.name;				// 取得author的"surname"属性
let title = book["main title"];		// 取得book的"main title"属性
```

创建或者设置属性：

```js
book.edition = 7;
book["main title"] = "ECMAScript";
```

### 作为关联数组的对象

```js
let add = "";
for (let i = 0; i < 4; i++) {
    addr += curtomer['address${i}'] + "\n";
}
```

JS对象经常像这样作为关联数组使用。

ES6之后，使用`Map`类通常比使用普通对象更好。

### 继承

假如要从对象`o`中查询属性`x`。如果`o`没有叫这个名字的自由属性，则会从`o`的原则对象查询属性`x`。如果原型对象也没有叫这个名字的自由属性，但它有自己的原型，那么就会继续查这个原型的原型。直至找到属性`x`或者查询到一个原型为`null`的对象。

```js
let o = {};					// o从Object.prototype继承对象方法
o.x = 1;					// 现在它有了自有属性x

let p = Object.create(o);	// p从o和Object.prototype继承属性
p.y = 2;					// 而且有一个自有属性y

let q = Object.create(p);	// q从p、o和Object.prototype继承属性
q.z = 3;					// 并且有一个自有属性z

let f = q.toString();		// toString继承自Object.prototype
q.x + q.y					// 3 x和y分别继承自o和p
```

属性赋值查询原型链只为确定是否允许赋值。

如果继承的是只读属性，那就不能赋值。

查询属性属性时候会用到原型链，而设置属性时不影响原型链是一个重要的JS特性，利用该特性，可以选择性地覆盖继承的属性。

### 属性访问错误

访问不存在的属性不是错误，访问不存在的属性会返回`undefined`。

但是访问不存在的属性的属性就是错误了，因为`null`和`undefined`就是错误。

```js
let len = book.subtitle.length;		// TypeError undefined没有length属性
```

解决方法：

```js
let surname = undefined;
if (book) {
    if (book.author) {
        surname = book.author.surname;
    }
}

surname = book && book.author && book.author.surname
surname = book?.author?.surname;
```

## 删除属性

```js
delete book.author;			// book对象现在没有author属性了
delete book["main title"];	// book对象现在没有"main title"属性了
```

`delete`只删除自有属性，不会删除继承属性。删除继承属性只能从原型对象上删除，但是那样会影响继承该原型的所有对象。

`delete`不会删除`configurable`特性为`false`的属性。

在非严格模式下删除全局对象可配置的属性时候，可以省略对全局对象的引用，直接使用属性名。

## 测试属性

`in`操作符要求左边是个属性名，右边是个对象。如果对象有包含相应名字的自由属性或继承属性，就返回`true`：

```js
let o = { x: 1 };
"x" in o			// true
"y" in o			// false
"toString" in o		// true
```

对象的`hasOwnProperty()`方法用于测试对象是否有给定名字的属性。对于继承的属性会返回`false`：

```js
o.hasOwnProperty("x")				// true
o.hasOwnProperty("y")				// false
o.hasOwnProperty("toString")		// false toString是继承属性
```

`propertyIsEnumberable()`方法细化了`hasOwnProperty()`测试：若传入的命名属性是自有属性且该属性的`enumerable`特性为`true`，则返回`true`。

```js
let o = {x: 1};
o.propertyIsEnumberable("x")						// true o有个可枚举属性x
o.propertyIsEnumberable("toString")					// false toString不是自有属性
Object.prototype.propertyIsEnumberable("toString")	// false toString不可枚举
```

通常简单的属性查询使用`!==`就行了：

```js
o.x !== undefined;
```

不过`in`可以区分不存在的属性和存在但是被设为`undefined`的属性：

```js
let o = {x: undefined};				// 把属性显式设为undefined
o.x !== undefined;					// false 属性x存在但是值是undefined
o.y !== undefined;					// false 属性y不存在
"x" in o				// true 属性x存在
"y" in o				// false 属性y不存在
delete o.x;				// 删除属性 x
"x" in o				// false 属性x不存在
```

## 枚举属性

我们为对象添加的属性默认是可以枚举的。

为了防止通过`for/in`枚举继承的属性，额可以在循环体内添加一个显式测试：

```js
for(let p in o) {
    if(!o.hasOwnProperty(p)) continue;			// 跳过继承属性
    if(typeof o[p] === "function") continue;	// 跳过所有方法
}
```

下述函数可以用于取得属性名数组：

1. `Object.keys()`：返回对象可枚举自有属性名的数组。不包含不可枚举属性和继承属性和名字是符号的属性。
2. `Object.getOwnPropertyNames()`：类似上面的，但会返回不可枚举自有属性名的数组，只要它们的名字是字符串。
3. `Object.getOwnPropertySymbols()`：返回名字是符号的自由属性，不管可不可以枚举。
4. `Reflect.ownKeys()`：返回所有属性名。

### 属性枚举顺序

ES6定义了枚举对象自有属性的顺序：

1. 先列出名字为非负整数的字符串属性，按照数值顺序从小到大。意味数组和类数组对象的属性会按照顺序被枚举。
2. 再列出所有剩下的字符串名字的属性。按照它们被添加到对象的先后顺序列出。 对于在对象字面量里的属性，按照他们在字面量里出现的顺序列出。
3. 最后名字为符号对象的属性按照它们被添加到对象的先后顺序列出。

`for/in`循环的枚举顺序不像上述枚举函数那么严格，但通常会按照上面描述的顺序枚举自有属性，然后再沿原型链上溯，以同样的顺序枚举每个原型对象的属性。但要注意：若已有同名属性被枚举过了，甚至若有一个同名属性是不可枚举的，则该属性就不会枚举了。

## 扩展对象

JS中，将一个对象的属性复制到另一个对象上是很常见的：

```js
let target = {x: 1}, source = {y: 2, z: 3};
for(let key of Object.keys(source)) {
    target[key] = source[key];
}
target			// {x: 1, y: 2, z: 3}
```

ES6中该能力被添加到JS里了：

```js
Object.assign(o, defaults);		// 用defaults覆盖o的所有属性
```

`Object.assign`接受两个或多个对象作为其参数。会修改并返回第一个参数，不会修改第二个及后续参数，第二个及后续参数都是来源对象。会把每个来源对象的可枚举自有属性复制到第一个参数对象上。第一个==来源==对象的属性会覆盖第一个参数对象上的属性，而第二个来源对象又会覆盖第一个来源对象上的同名属性。

```js
o = Object.assign({}, defaults, o);
```

为避免额外的对象创建和复制，也可以重写一个只复制那些不存在的属性：

```js
function merge(target, ...sources) {
    for(let source of sources) {
        for(let key of Object.Keys(source)) {
            if(!key in targe) {
                target[key] = source[key];
            }
        }
    }
    return target;
}

Object.assign({x: 1}, {x: 2, y: 2}, {y: 3, z: 4})		// {x: 2, y: 3, z: 4}
merge({x: 1}, {x: 2, y: 2}, {y: 3, z: 4})				// {x: 1, y: 2, z: 4}
```

## 序列化对象

对象序列化(serialization)是将对象的状态转换为字符串的过程，之后可以从中恢复对象的状态。

- `JSON.stringify()`：序列化对象。仅序列化对象的可枚举自由属性。若属性值无法序列化，该属性会从输出的字符串中删除。
- `JSON.parse()`：恢复对象。

上面两个方法都接受可选的第二个参数，用于自定义序列化及恢复操作。详细内容在第十一章。

JSON(JavaScript Object Notation，Javascript对象表示法)，语法：

```js
let o = {x: 1, y: {z: [false, null, " "]}};		// 定义一个测试对象
let s = JSON.stringify(o);			// s = '{"x": 1, "y": {"z": [false, null, " "]}}'
let p = JSON.parse(s);				// p = {x: 1, y: {z: [false, null, " "]}}
```

可以序列化和恢复的值包括：

1. 对象
2. 数组
3. 字符串
4. 有限数值
5. `true`
6. `false`
7. `null`

`NaN`和`Infinity`和`-Infinity`会被序列化为`null`。

日期对象会被序列化为ISO格式的日期字符串(`Date.toJSON()`)，但`JSON.parse()`会保持它的字符串形式，不会恢复原始的日期对象。

## 对象方法

### toString()方法

不接受参数，返回表示它的对象的值的字符串。

每当需要把一个对象转为字符串时，JS都会调用该对象的这个方法。 

可以像这样定义自己的`toString`：

```js
let point = {
    x: 1,
    y: 2,
    toString: function() { return '(${this.x}, ${this.y})'; }
}
String(point)			// "(1, 2)"
```

### toLocaleString()方法

除`toString`，对象也有个`toLocaleString()`方法。用于返回对象的本地化字符串表示。

`Object`定义的默认`toLocaleString()`方法没有实现任何本地化，而是调用`toString()`并返回该值。

`Date`和`Number`类定义了自己的`tolocaleString`方法，尝试根据本地管理格式化数值、日期和时间。

数组也有`toLocaleString`方法，会调用每个元素的`toLocaleString`方法。

### valueOf()方法

`valueOf`会在JS需要把对象转换为某些非字符串原始值(通常是数值)时被调用。

一些内置类定义了自己的`valueOf()`方法。`Date`类定义的`valueOf()`方法可以将日期转换为数值。

### toJSON()方法

`Object.prototype`实际上没有定义`toJSON()`方法，但`JSON.stringify()`会从要序列化的对象上寻找`toJSON()`方法。若存在，则调用它，然后序列化该方法的返回值，而非原始对象。

`Date`类定义了自己的`toJSON`方法，返回一个表示日期的序列化字符串。

也能够自定义该方法：

```js
let point = {
    x: 1,
    y: 2,
    toString: function() { return '(${this.x}), ${this.y})'; },
    toJSON: function() { return this.toString(); }
}

JSON.stringfly([point])			// '["(1, 2)"]'
```

## 对象字面量扩展语法

### 简写属性

```js
// 原来
let x = 1, y = 2;
let o = {
    x: x,
    y: y
}

// 等同上方
// ES6之后
let x = 1, y = 2;
let o = { x, y };
```

### 计算的属性名

创建一个具有特定属性的对象，但该属性的名字不是编译时可以直接写在源代码中的常量。

```js
const PROPERTY_NAME = "p1";
function computePropertyName() { return "p" + 2; }

let o = {};
o[PROPERTY_NAME] = 1;
o[computePropertyName()] = 2;
```

使用ES6称为计算属性的特性可以更简单地创建类似对象：

```js
const PROPERTY_NAME = "p1";
function computePropertyName() { return "p" + 2; }

let p = {
    [PROPERTY_NAME]: 1,
    [computePropertyName()]: 2
};

p.p1 + p.p2		// 3
```

可能用到的场景：JS代码库，需要给该库传入一个包含一组特定属性的对象，而该属性的名字在该库中是以常量形式定义的。如果通过代码来创建要传给该库的这个对象，可以硬编码它的属性名，但是这样有可能把属性名写错，同时也存在因为库版本升级而修改了属性名导致的错配问题。

通过计算属性语法来创建对象会让我们的代码更加可靠。

### 符号作为属性名

ES6后，符号可以是字符串或符号。

```js
const extension = Symbol("my extension symbol");
let o = {
    [extension]: {}
};

o[extension].x = 0
```

第三章已经所过，符号是不透明值，除了作为属性名外，不能用它们做任何事情。

使用符号不是为了安全，而是为JS对象定义安全的扩展机制。如果从不受控的第三方代码得到一个对象，然后需要给该对象添加一些自己的属性，但又不希望与原对象内属性有冲突，就可以放心的用符号作为属性名。

不过第三方代码也可以用`Object.getOwnPropertySymbols()`找到我们所使用的符号，然后修改或删除我们的属性。这也是符号不是一种安全机制的原因。

### 扩展操作符

ES2018后，可以在对象字面量中使用"扩展操作符"将已有对象的属性复制到新对象中：

```js
let position = {x: 0, y: 0};
let dimensions = {width: 100, height: 75};
let rect = { ...position, ...dimensions};
rect.x + rect.y + rect.width + rect.height		// 175
```

`...`仅在对象字面量中有效。

如果扩展对象和被扩展对象有同名属性，那么同名属性值由后面的对象决定：

```js
let o = {x: 1}
let p = {x: 0, ...o}
p.x			// 1 使用o的
let q = { ...o, x: 2}
q.x			// 2 使用q自带的
```

它可能给JS解释器带来巨大的工作量。若对象有`n`个属性，将该属性扩展到另一个对象可能是种`O(n)`操作。意味着，如果在循环或递归函数中通过`...`向一个大对象不断追加属性，那么很可能是在写一个低效的$O(n^2)$算法。随着`n`越来越大，该算法可能称为性能瓶颈。

### 简写方法

把函数定义为对象属性时，称该函数为方法。

ES6前，需要像定义对象的其他属性一样，通过函数定义表达式在对象字面量中定义一个方法：

```js
let square = {
    area: function() { return this.side * this.side; }
    side: 10
};

square.area()			// 100
```

但ES6后，对象字面量语法经过扩展，允许省略`function`关键字和冒号的简写方法：

```js
let square = {
    area(){ return this.side * this.side; }
    side: 10
};
square.area()			// 100
```

也可以使用字符串字面量和计算的属性名，包括符号属性名：

```js
const METHOD_NAME = "m";
const symbol = Symbol();
let weirdMethods = {
    "method With Spaces"(x) { return x + 1; },
    [METHOD_NAME](x) { return x + 2; },
    [symbol](x) { return x + 3; }
};

weirdMethods["method With Spaces"](1)		// 2
weirdMethods[METHOD_NAME](1)				// 3
weirdMethods[symbol](1)						// 4
```

为了让对象可以迭代(以便在`for/of`循环中使用)，必须以符号名`Symbol.iterator`给它定义一个方法。

### 属性的获取方法与设置方法

除了数据属性，JS还支持为对象定义访问器属性(accessor property)，这种属性不是一个值，而是一个或两个访问器方法：

- 一个获取方法(getter)
- 一个设置方法(setter)

访问器属性定义(ES5引入)：

```js
let o = {
    dataProp: value,
    
    get accessorProp() { return this.dataProp; },
    set accessorProp() { this.dataProp = value; }
}
```

ES6中，也可以使用计算的属性名来定义获取方法和设置方法。只要把`get`和`set`后面的属性名替换为用方括号包含的表达式就行。

表示2D笛卡尔坐标点的对象。这个对象用普通数据属性保存点的`x`和`y`坐标，用访问器属性给出与该点等价的极坐标：

```js
let p = {
    x: 1.0,
    y: 1.0,
    
    // r是由获取方法和设置方法定义的可读写访问器属性
    get r() { return Math.hypot(this.x, this.y); },
    set r(newvalue) {
        let oldvalue = Math.hypot(this.x, this.y);
        let ratio = newvalue/oldvalue;
        this.x *= ratio,
       	this.y *= ratio,
    }c,
        
    // theta是一个只定义了获取方法的只读访问器属性
    get theta() { return Math.atan2(this.y, this.x); }
}
```

类似数据属性，访问器属性也是可以继承的。所以可以把上面定义的对象`p`作为其它点的原型。可以给新对象定义自己的`x`和`y`属性，而它们将继承`r`和`theta`属性：

```js
let q = Object.create(p);
q.x = 3; q.y = 4;	// 创建q的自有数据属性
q.r					// 5
q.theta				// Math.atan2(4, 3)
```

使用访问器属性额其他场景还有写入属性时进行合理性检查，以及每次读取属性时返回不同的值：

```js
const serialnum = {
    // 该属性属性保存下一个序号
    _n: 0,
    get next() { return this._n++; },
    
    // 将新值设为n 但n必须大于当前值
    set next(n) {
        if (n > this._n) this._n = n;
        else throw new Error("serial number can only be set to a larger value");
    }
};

serialnum.next = 10			// 设置起始序号
serialnum.next				// 10
serialnum.next				// 11
```

