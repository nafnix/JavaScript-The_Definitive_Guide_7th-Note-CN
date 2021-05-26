# 元编程

如果说常规编程是写代码去操作数据，那么元编程就是写代码区操作其他代码。

## 属性的特性

JS的属性除名字和值还有3个关联的特性，用于指定属性的行为以及可以对其执行的操作：

1. 可写(writable)特性：是否可以修改属性的值
2. 可枚举(enumerable)特性：可以通过`for/in`循环和`Object.keys()`方法枚举属性
3. 可配置(configurable)特性：
   - 是否可以删除属性
   - 是否可以修改属性

对象字面量定义的属性或者通过常规赋值方式给对象定义的属性都可写、可枚举、可配置。但JS标准库中定义的很多属性并非如此。

通过本节讲解与设置属性的特性有关的API：

- 该API允许库作者给原型对象添加方法，并让它们像内置方法一样不可枚举
- 该API允许作者"锁住"自己的对象，定义不能修改或删除的属性

本节将访问器属性的获取方法和设置方法作为属性的特性来看待。按该逻辑，甚至也把数据属性的值当成一个特性。即一个属性有一个名字和四个特性。

用于查询和设置属性特性的JS方法使用属性描述符(property descriptor)对象，这个对象用于描述属性的4个特性。

属性描述符对象拥有(它所(描述的属性)的特性)属性名。

即数据属性的属性描述符有如下属性：

- `value`
- `writable`
- `enumerable`
- `configurable`

访问器属性的属性描述符没有`value`和`writable`，只有`get`和`set`属性。其中`writable`、`enumerable`、`configurable`属性是布尔值，`get`和`set`是函数。

`Object.getOwnPropertyDescriptor()`可以获得特定对象某个属性的属性描述符：

```js
// { value: 1, writable: true, enumerable: true, configurable: true }
Object.getOwnPropertyDescriptor({x: 1}, "x");

// 该对象只有一个只读的访问器属性
const random = {
    get octet() { return Math.floor(Math.random() * 256); }
};

// 返回 { get: /*func*/ , set:undefined, enumerable:true, configurable: true}
Object.getOwnPropertyDescriptor(random, "octet"); 

// 对继承的属性或不存在的属性返回undefined
Object.getOwnPropertyDescriptor({}, "x")			// undefined 没有该属性
Object.getOwnPropertyDescriptor({}, "toString")		// undefined 继承的属性
```

`Object.getOwnPropertyDescriptor()`只对自有属性有效。

要设置属性的特性或要创建一个具有指定特性的属性，可以调用`Object.defineProperty()`方法，传入：

1. 要修改的对象
2. 要创建或修改的属性的名字
3. 属性描述符对象

```js
let o = {};		// 一开始没有任何属性

// 创建一个不可枚举的数据属性x 值为1
Object.defineProperty(o, "x", {
    value: 1,
    writable: true,
    enumerable: false,
    configurable: true
});

// 确认已有该属性 且不可枚举
o.x					// 1
Object.keys(o)		// []


// 修改属性x为只读
Object.defineProperty(o, "x", { writable: false });

// 修改属性值
o.x = 2			// 静默失败或在严格模式下抛出TypeError
o.x				// 1

// 该属性仍然可以配置 所以可以像这样修改其值
Object.defineProperty(o, "x", { value: 2 });
o.x					// 2

// 将x由数据属性转为访问器属性
Object.defineProperty(o, "x", { get: function(){ return 0; } });
o.x					// 0
```

若是创建新属性，那么省略的特性会取得`false`或`undefined`值。

若是修改已有属性，那么省略的特性就不会修改。

只会修改或创建已有属性，不会修改继承属性。

`Object.defineProperties()`：一次创建或修改多个属性，参数如下：

1. 要修改的对象
2. 创建或修改的属性的名称映射到这些属性的属性描述符对象

```js
let p = Object.defineProperties({}, {
    x: { value: 1, writable: true, enumerable: true, configurable: true },
    y: { value: 1, writable: true, enumerable: true, configurable: true },
    r: {
        get() { return Math.sqrt(this.x*this.x + this.y*this.y); },
        enumerable: true,
        configurable: true
    }
});
p.r		// Math.SQRT2
```

为空对象添加两个数据属性和一个只读的访问器属性。

给`Object.create()`传入一组属性描述符，可以给新创建的对象添加属性。

规则：

- 若对象不可扩展，可以修改其已有属性，但不能给它添加新属性
- 若属性不可配置，不能修改其`configurable`或`enumerable`特性
- 若访问器属性不可配置，不能修改其获取方法或设置方法，也不能将其修改为数据属性
- 若数据属性不可配置：
  - 不能将其修改为访问器属性
  - 不能将它的`writable`特性由`false`改为`true`，但可以由`true`改为`false`
- 若数据属性：
  - 不可配置且不可写：不能修改其值
  - 可配置但不可写：可修改其值(先将其配置为可写，再修改其值，再将其配置为不可写)

第六章说过的`Object.assign()`只赋值可枚举属性和属性值，但不复制属性特性，这也意味如果源对象有个访问器属性，那么复制到目标对象的是获取函数的返回值，而非获取函数本身。

自定义一个复制全部属性描述符的函数：

```js
Object.defineProperty(Object, "assignDescriptors", {
    // 与调用Object.assign()时的特性保持一致
    writable: true,
    enumerable: false,
    configurable: true,
    // 这个函数是assignDescriptors属性的值
    value: function(target, ...sources) {
        for(let source of sources) {
            for(let name of Object.getOwnPropertyNames(source)) {
                let desc = Object.getOwnPropertyDescriptor(source, name);
                Object.defineProperty(target, name, desc);
            }
            
        	for(let symbol of Object.getOwnPropertySymbols(source)) {
            	let desc = Object.getOwnPropertyDescriptor(source, symbol);
            	Object.defineProperty(target, symbol, desc);
        	}
        }
        return target;
    }
});


let o = {c: 1, get count(){return this.c++;}};		// 定义包含获取方法的对象
let p = Object.assign({}, o);						// 复制属性的值
let q = Object.assignDescriptors({}, o);			// 复制属性描述符

p.count				// 1 只是一个数据属性 因此它的值不会递增
p.count				// 1

q.count				// 2 复制的时候就会递增 复制的是获取方法 因此每次访问都会递增
q.count				// 3
```

## 对象的可扩展能力

对象的可扩展(extensible)特性控制是否可以给对象添加新属性，即是否可扩展。

普通JS对象默认可扩展。

修改不可扩展对象的原型始终抛出`TypeError`。

将对象改为不可扩展是不可逆的。

extensible特性的作用是将对象"锁定"在已知状态，阻止外部篡改。

- `Object.isExtensible()`：确定对象是否可扩展。
- `Object.preventExtensions()`：让对象不可扩展。
  - 若再对该对象添加属性：
    - 严格模式：抛出`TypeError`
    - 非严格模式：静默失败
  - 只影响对象本身可扩展能力。如果给一个不可扩展对象的原型添加了新属性，那么这个不可扩展对象还是会继承这些新属性。
- `Object.seal()`：类似`Object.preventExtensions()`对象不可扩展并且对象的所有自有属性不可扩展。也就是不能给对象添加新属性，也不能删除或配置已有属性。但可写的已有属性依然可写。无法"解封"已被"封存"的对象。
- `Object.isSealed()`：确认对象是否被封存。
- `Object.freeze()`：除了让对象不可扩展，让其属性不可配置，还将对象的全部自由属性变成只读。但若对象有访问器属性，且访问器属性有设置方法，则这些属性不受影响，还是可以调用它们给属性赋值。
- `Object.isFrozen()`：确认对象是否被冻结。

`.seal()`和`.freeze()`只影响传给自己的对象，不影响对象原型。若想彻底锁定一个对象，那可能也要封存或冻结其原型链上的对象。

`.preventExtensions()`和`.seal()`和`.freeze()`都返回传给它们的对象，意味可以在嵌套函数中使用：

```js
// 创建一个原型被冻结的封存对象，有个不可枚举的自由属性
let o = object.seal(Object.create(Object.freeze({x: 1}), {y: {value: 2, writable: true}}));
```

被冻结的对象可能影响常规的JS测试策略。

## prototype特性

对象的`prototype`特性指定对象从何处继承属性。

当`prototype`以代码字体出现时，指普通对象的属性，而非`prototype`特性。

对象的`prototype`特性是在对象被创建时设定的。使用对象字面量创建的对象使用`Obejct.prototype`作为其原型。用`new`创建的对象用构造函数的`prototype`属性的值作为其原型。用`Object.create()`用传给它的第一个参数作为原型。

`Object.getPrototypeOf()`：查询对象原型

```js
Object.getPrototypeOf({})			// Object.prototype
Object.getPrototypeOf([])			// Array.prototype
Object.getPrototypeOf(()=>{})		// Function.prototype
```

`isPrototypeOf()`：确定一个对象是不是另一个对象的原型(或原型链中的一环)

```js
let p = {x: 1};						// 定义一个原型对象
let o = Object.create(p)			// 用该原型创建一个对象
p.isPrototypeOf(o)					// true o继承p
Object.prototype.isPrototypeOf(p)	// true p继承Object.prototype
Object.prototype.isPrototypeOf(o)	// true o也继承Object.prototype
```

`isPrototypeOf()` 类似`instanceof`。

对象的`prototype`特性在创建时被设定，通常不会再改。

`Object.setPrototypeOf()`：修改对象原型

```js
let o = {x: 1};
let p = {y: 2};
Object.setPrototypeOf(o, p);		// 将o的原型改为p
o.y									// 2 继承属性y
let a = [1, 2, 3];
Object.setPrototypeOf(a, p);		// 将数组a的原型设置为p
a.join								// undefined a不再有join()方法
```

JS实现可能会基于对象原型固定不变的假定实现激进的优化。也就是如果调用`setPrototypeOf()`可能使得被修改的对象比正常情况下慢。

JS一些早期浏览器实现通过`__proto__`属性暴露对象的`prototyep`特性。该属性再很早前就被废弃了，但网上还有很多代码依赖`__proto__`。ECAMScript标准为此为要求所有浏览器的JS实现都必须支持它。

现代JS中，`__proto__`是可读可写的，可以但不该用它代替`Object.getPrototypeOf()`和`Object.setPrototypeOf()`。

用`__proto__`修改对象字面量的原型：

```js
let p = {z: 3};
let o = {
    x: 1,
    y: 2,
    __proto__: p
};
o.z		// 3 o继承p
```

## 公认符号

之所以添加`Symbol`类型，主要是为了便于扩展JS，同时不会破坏对Web上已有代码的向后兼容性。

`Symbol.iterator`是最为人熟知的"公认符号"(well-known symbol)。

公认符号：`Symbol()`工厂函数的一组属性，即一组符号值。通过这些符号值，可以控制JS对象和类的某些底层行为。

### Symbol.iterator和Symbol.asyncIterator

`Symbol.iterator`和`Symbol.asyncIterator`符号可以让对象或类将自己变成可迭代对象和异步可迭代对象。

### Symbol.hasInstance

ES6中，若`instanceof`的右侧是个有`[Symbol.hasInstance]`方法的对象，那么就会以左侧的值作为参数来调用该方法并返回该方法的值，返回值会转为布尔值，变成`instanceof`操作符的值。

可以用`instanceof`操作符对适当定义的伪类型对象去执行通用类型检查：

```js
// 定义一个作为"类型"的对象 以便与instanceof一起使用
let uint8 = {
    [Symbol.hasInstance](x) {
        return Number.isInteger(x) && x >= 0 && x <= 255;
    }
};

128 instanceof uint8		// true	{[Symbol.hasInstance](128)}
256 instanceof uint8		// false 太大
Math.PI instanceof uint8	// false 非整数
```

### Symbol.toStringTag

调用一个简单JS对象的`toString()`方法能得到`"object Object"`;

```js
{}.toString()		// "[object Object]"
```

如果调用与内置类型实例的方法相同的`Object.prototype.toString()`则：

```js
Object.prototype.toString.call([])			// "[object Array]"
Object.prototype.toString.call(/./)			// "[object RegExp]"
Object.prototype.toString.call(false)		// "[object Boolean]"
```

使用`Object.prototype.toString().call()`检查任何JS值，都能从包含类型信息的对象中获取以其他方式无法获取的"类特性"。

而下面该`classof()`比`typeof`操作符更有用，因为`typeof`无法区分不同对象的类型：

```js
function classof(o) {
    return Object.prototype.toString.call(o).slice(8, -1);
}

classof(null)			// "Null"
classof(new Date())		// "Date"
```

ES6前，`Object.prototype.toString()`这种特殊用法只对内置类型的实例有效。若对自定义的类的实例调用`classof()`，只能得到`"Object"`。

ES6中，它会查找自己参数中有没有一个属性的符号名是`Symbol.toStringTag`。若有，则输出该属性值。意味若自己定义了一个类，可以很容易让其适配`classof()`这样的函数：

```js
class Range {
    get [Symbol.toStringTag]() { return "Range"; }
}

let r = new Range(1, 10);
Object.prototype.toString.call(r)		// "[object Range]"
classof(r)								// "Range"
```

### Symbol.species

ES6前，JS没提供任何实际方式去创建内置类的子类。

ES6中，可以用`class`和`extends`扩展任何内置类。

- ES6及之后，`Array()`构造函数有个名为`Symbol.species`符号属性
- 使用`extends`创建子类时，子类构造函数会从父类构造函数继承属性。也就是`Array`的每个子类的构造函数都会继承`Symbol.species`
- ES6及之后，`map()`和`slice()`等创建并返回新数组的方法修改为不仅会创建一个常规`Array`，还会调用`new this.constructor[Symbol.species]()`创建新数组

ES6中`Array[Symbol.species]`是个只读的访问器属性，其获取函数返回`this`。子类构造函数继承该获取函数，也就是每个子类构造函数都是其自己的"物种"(species)。

但如果像修改该默认行为，比如让继承`Array`的子类返回常规的`Array`对象，那就只要将`Array子类[Symbol.species]`改为`Array`即可，为此可以用`defineProperty()`。假设有个`EZArray`继承`Array`：

```js
EZArray[Symbol.species] = Array;		// 设置只读属性会失败

// 使用defineProperty()
Object.defineProperty(EZArray, Symbol.species, { value: Array });
```

最简单就是在定义`EZArray`时就定义自己的`Symbol.species`：

```js
class EZArray extends Array {
    static get [Symbol.species]() { return Array; }
    get first() { return this[0]; }
    get last() { return this[this.length - 1]; }
}

let e = new EZArray(1, 2, 3);
let f = e.map(x => x - 1);

e.last		// 3
f.last		// undefined f是个常规数组 没有last获取方法
```

定性数组也以同样的方式使用了这个符号。

`ArrayBuffer`的`slice()`方法也会查找`this.constructor`的`Symbol.species`属性。

返回新`Promise`对象的方法(如`then()`)也通过该"物种协议"创建返回的期约。

### Symbol.isConcatSpreadable

ES6中，若`concat()`的参数(或`this`值)是对象且有个`Symbol.isConcatSpreadable`符号属性，则根据该属性的布尔值来确定是否该"展开"参数。若不存在，则像ES6前一样使用`Array.isArray()`确定是否该将某个值作为数组对待。

两种情况下可能用到该`Symbol`：

1. 创建一个类数组对象，且希望将其传给`concat()`时该对象能像真正的数组一样，可以给该对象添加这么个符号属性：

   ```js
   let arraylike = {
       length: 1,
       0: 1,
       [Symbol.isConcatSpreadable]: true
   };
   
   [].concat(arraylike)			// [1] (若不展开 此处就是[[1]])
   ```

2. `Array`的子类默认是可展开的，若定义了个数组的子类，但不希望其在传给`concat()`时像数组一样，可以像下面这样给该子类添加一个获取方法：

   ```js
   class NonSpreadableArray extends Array {
       get [Symbol.isConcatSpreadble]() { return false; }
   }
   
   let a = new NonSpreadableArray(1, 2, 3);
   [].concat(a).length			// 1 (如果a被展开了 这里就是3个元素)
   ```

### 模式匹配符号

ES6之后，使用正则参数执行模式匹配操作的String方法都统一泛化为既能用正则对象，也能使用任何通过具有符号名的属性定义了模式匹配行为的对象。

`match()`、`matchAll()`、`search()`、`replace()`和`split()`这些字符串方法都有与之对应的公认符号：`Symbol.match`、`Symbol.search`等。

有了泛化之后的字符串方法，就可以用公认的符号方法定义自己的模式类，提供自定义匹配。

一般在像下面这样调用上面5个字符串方法时：

```js
string.method(pattern, arg)
```

会转换为对模式对象上相应符号化命名方法的调用：

```js
pattern[symbol](string, arg)
```

示例：

```js
class Glob {
    constructor(glob) {
        this.glob = glob;
        
        // 内部使用RegExp实现glob匹配
        // ?匹配除/之外的任意字符 *匹配0或多个这样的字符
        // 每个匹配都用一个捕获组
        let regexpText = glob.replace("?", "([^/])").replace("*", "([^/]*)");
        
        // 使用u标志表示支持Unicode的匹配
        // glob是要匹配整个字符串 所以用^和$锚点
        // 这里没实现search()和match() 因为它们对这样的模式没有用
        this.regexp = new RegExp(`^${regexpText}`, "u");

    }
    
    toString() { return this.glob; }
    
    [Symbol.search](s) { return s.search(this.regexp); }
    [Symbol.match](s) { return s.match(this.regexp); }
    [Symbol.replace](s, replacement) { return s.replace(this.regexp, replacement); }
}


let pattern = new Glob("docs/*.txt");
"docs/js.txt".search(pattern);		// 0 从第0个字符开始匹配
"docs/js.htm".search(pattern);		// -1 不匹配

let match = "docs/js.txt".match(pattern);
match[0]			// "docs/js.txt"
match[1]			// "js"
match.index			// 0
"docs/js.txt".replace(pattern ,"web/$1.htm")			// "web/js.htm"
```

### Symbol.toPrimitive

第三章解释过JS有3个稍微不同而算法，用于将对象转为原始值。

ES6中，公认符号`Symbol.toPrimitive`允许我们覆盖这个默认的对象到原始值的转换行为，让我们完全控制自己的类实例如何转为原始值。

首先定义一个名字为该符号的方法，该方法要接收一个能够表示对象的原始值。该方法在被调用时会收到一个字符串参数，用于告诉我们JS打算对我们的对象做什么样的转换：

- 若参数是`"string"`：表示JS是在一个预期或偏好为字符串的上下文中进行转换。
- 若参数是`"number"`：表示JS是在一个预期或偏好为数值的上下文中进行转换。常出现于比较操作符和算术操作符中。
- 若参数是`"default"`：表示JS做该转换的上下文可以接受数值也可以接受字符串。

### Symbol.unscopables

这是针对`with`语句所导致的兼容性问题引入的一个方案。

ES6及之后，`with`语句被改为在取得对象`o`时，`with`语句会计算`Object.keys(o[Symbol.unscopables]||{})`，并在创建用于执行语句体的模拟作用域时，忽略名字包含在结果数组中的哪些属性。

ES6使用该机制给`Array.prototype`添加新方法，同时不会破坏线上已有的代码。

即可以通过如下方式获取最新`Array`方法的列表：

```js
let newArrayMethods = Object.keys(Array.prototype[Symbol.unscopables]);
```

## 模板标签

