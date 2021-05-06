# 类

JS中，类使用基于原型的继承。

JS的类和基于原型的继承机制与JAVA等语言中类和基于类的继承机制有着本质区别。

## 类和原型

JS中，类意味着一组对象从一个原型对象继承属性。

如果我们定义了一个原型对象，然后用`Object.create()`创建一个继承它的对象，那我们就定义了一个JS类。

下述代码定义了一个函数来创建和初始化新对象：

```js
function range(from, to) {
    let r = Object.create(range.methods);
    r.from = from;
    r.to = to;
    return r;
}

range.methods = {
    includes(x) { return this.from <= x && <= this.to; },
    *[Symbol.iterator]() {
        for(let x = Math.ceil(this.from); x <= this.to; x++) yield x;
    },
    toString() { return "(" + this.from + "..." + this.to + ")"; }
};

let r = range(1, 3);		// 创建一个范围对象
r.includes(2)				// true 2在范围内
r.toString()				// "(1...3)"
[...r]						// [1, 2, 3]
```

## 类和构造函数

构造函数要用`new`调用。构造函数调用的关键在于构造函数的`prototype`属性将被用作新对象的原型。

只有函数对象才有`prototype`属性。这意味着使用同一个构造函数创建的所有对象都继承同一个对象。

下述代码演示了在不支持ES6`class`的JS版本中创建类的习惯做法：

```js
function Range(from, to) {
    this.from = from;
    this.to = to;
}

// 必须命名为prototype
Range.prototype = {
    includes: function(x) { return this.from <= x && x <= this.to; },
    [Symbol.iterator]: function*() {
        for(let x = Math.ceil(this.from); x <= this.to; x++) yield x;
    },
    toString: function() { return "(" + this.from + "..." + this.to + ")"; }
};

let r = new Range(1, 3)			// 创建一个Range对象
r.includes(2)			// true 2在范围内
r.toString()			// "(1...3)"
[...r]					// [1, 2, 3]
```

构造函数调用与普通函数调用的这个重要区别也是我们用首字母大写的名字命名构造函数的一个原因。

这个命名规定让构造函数有别于普通函数，方便程序员知道什么时候使用`new`。

#### 构造函数和new.target

函数体可以通过特殊表达式`new.target`判断函数是否作为构造函数被调用了。

JS的各种错误构造函数可以不使用`new`调用。

不过该技术只适用于以老方式定义的构造函数。使用`class`关键字创建的类不允许不使用`new`调用它们的构造函数。

`Range.prototype`这个名字是强制性的。对`Range()`的调用会自动把`Range.prototype`作为新`Range`对象的原型。

用箭头函数方式定义的函数没有`prototype`属性，所以不能作为构造函数来用。

### 构造函数、类标识和instanceof

原型对象是类标识的基本：当且仅当两个对象继承同一个原型对象时，它们才是同一个实例。

```js
function Strange() {}
Strange.prototype = Range.prototype;
new Strange() instanceof Range;		// true
```

若不想以构造函数作为媒介，直接测试某个对象原型链中是否包含指定原型，可以用`isPrototypeOf()`。

例如在前面的`range.methods`中没有使用构造函数，所以无法对该类使用`instanceof`操作符。此时可以通过如下代码检测对象`r`是不是这个无构造函数类的成员：

```js
range.methods.isPrototypeOf(r);				// range.methods是r的原型对象
```

### constructor属性

每个JS函数自动拥有一个`prototype`属性。该属性的值是一个对象，有一个不可枚举的`constructor`属性。而该`constructor`属性的值就是该函数对象：

```js
let F = function() {};		// 这是个函数对象
let p = F.prototype;		// 这是个与F关联的原型对象
let c = p.constructor;		// 与原型关联的函数
c === F		// true 对于任何F，F.prototype.constructor === F
```

在`Range`中，由于重写了`Range.prototype`对象。而它定义的这个新的原型对象并没有`constructor`属性。所以按照定义，`Range`类的实例没有`constructor`属性。该问题可以通过显式地为原型添加一个`constructor`属性来解决：

```js
Range.prototype = {
    constructor: Range,			// 显式设置反向引用constructor
    // 方法定义
};
```

另一个在老代码中常见的技术是：使用预定义的原型对象及其`constructor`属性，然后像下面这样每次给它添加一个方法：

```js
Range.prototype.includes = function(x) {
    return this.from <= x && x <= this.to;
};

Range.prototype.toString = function() {
    return "(" + this.from + "..." + this.to + ")";
};
```

## 使用class关键字的类

ES6引入。

使用`class`重写`Range`类：

```js
class Range {
    constructor(from, to) {
        this.from = from;
        this.to = to;
    }
    
    includes(x) { return this.from <= x && x <= this.to; }
    
    *[Symbol.iterator]() {
        for(let x = Math.ceil(this.from); x <= this.to; x++) yield x;
    }
    
    toString() { return `(${this.from}...${this.to})`; }
}

let r = new Range(1, 3)
r.includes(2)			// true
r.toString()			// "(1...3)"
[...r]					// [1,2,3]
```

使用`Range.prototype`定义的`Range`与`class`完全相同。

关键字`constructor`用于定义类的构造函数。但实际定义的函数不叫"constructor"。`class`声明语句会定义一个新的`Range`变量，并将该特殊构造函数的值赋给该变量。

如果类不需要任何初始化，那么可以省略`constructor`，解释器会隐式合成空构造函数。

类声明：

```js
let Square = class { constructor(x) { this.area = x * x; } };
new Square(3).area		// 9
```

类定义表达式也可以包含可选的类名。如果提供了名字，则该名字名字只能在类体内部访问到。

`class`声明体里所有代码默认处于严格模式。

类声明不会被提升，不能在声明类之前初始化它。

### 静态方法

静态方法是作为构造函数而非原型对象的属性定义的。

```js
static parse(s) {
    let matches = s.match(/^\((\d+)\.\.\.(\d+)\)$/);
    if (!matches) {
        throw new TypeError(`Cannot parse Range from "${s}".`)
    }
    return new Range(parseInt(matches[1]), parseInt(matches[2]));
}
```

这段代码定义的方法是`Range.parse()`，而非`Range.prototype.parse()`，必须通过构造函数而非实例调用它：

```js
let r = Range.parse('(1...10)');			// 返回一个新Range对象
r.parse('(1...10)');						// TypeError r.parse不是一个函数
```

因为静态方法是在构造函数上调用而非实例上，所以在静态方法里使用`this`没什么意义。

### 获取方法、设置方法及其他形式的方法

参见 第六章的属性的获取方法与设置方法，唯一区别是不用逗号分隔。

### 公有、私有和静态字段

本节代码可能在部分浏览器中还不受兼容。

原构造函数：

```js
class Buffer {
    constructor() {
        this.size = 0;
        this.capacity = 4096;
        this.buffer = new UintArray(this.capacity);
    }
}
```

可能被标准化的新写法：

```js
class Buffer {
    size = 0;
    capacity = 4096;
    buffer = new UintArray(this.capacity);
}
```

若要引用这些属性仍然要加上`this`。

在试图标准化该提案时也定义了私有属性，在属性名前加上`#`，则该属性只能在类体内带着`#`使用。

```js
class Buffer {
    #size = 0;
    get size(){return this.#size; }
}
```

私有属性必须先声明才能用。

新提案：若在公有或私有属性声明前加上`static`，这些字段就会创建为构造函数的属性，而非实例属性。例如前面创建的`Range.parse()`：

```js
static integerRangePattern = /^\((\d+)\.\.\.(\d+)\)$/
static parse(s) {
    let matches = s.match(Range.integerRangePattern);
    if (!matches) {
        throw new TypeError(`Cannot parse Range from "${s}".`)
    }
    return new Range(parseInt(matches[1]), parseInt(matches[2]));
}
```

## 为已有类添加方法

```js
// 为Complex类添加方法
Complex.prototype.conj = function() { return new Complex(this.r, -this.i); };

// 为String类添加方法
if(!String.prototype.startsWith) {
    String.prototype.startsWith = function(s) {
        return this.indexOf(s) === 0;
    };
}
```

像这样给内置类型的原型添加方法通常被认为是不好的做法。

给`Object.prototype`添加方法也是可以的，但是若这样做，那么所有对象都会继承该方法。

## 子类

ES6前做法。

定义`Range`类的子类`Span`：

```js
// Span类构造函数
function Span(start, span) {
    if (span >= 0) {
        this.from = start;
        this.to = start + span;
    } else {
        this.to = start;
        this.from = start + span;
    }
}


// 确保Span的原型继承Range的原型
Span.prototype = Object.create(Range.prototype);			// 继承


// 定义自己的constructor属性
Span.prototype.constructor = Span;

// 通过定义自己的toString方法
Span.prototype.toString = function() {
    return `(${this.from}... +${this.to - this.from})`;
}
```

### 通过extends和super创建子类

```js
// Array的一个子类
class EZArray extends Array {
    get first() { return this[0]; }
    get last() { return this[this.length-1]; }
}

let a = new EZArray();
a instanceof EZArray			// true
a instanceof Array				// true
a.push(1,2,3,4)					// a.length == 4
a.first							// 1
```

使用`super`调用父类构造函数和方法：

```js
class TypedMap extends Map {
    constructor(keyType, valueType, entries) {
        if(entries) {
            for (let [k, v] of entries) {
                if (typeof k !== keyType || typeof v !== valueType) {
                    throw new TypeError(`Wrong type for entry [${k}, ${v}]`);
                }
            }
        }
        
        // 初始化父类
        super(entries);
        
        // 初始化子类
        this.keyType = keyType;
        this.valueType = valueType;
        
        // 重定义set方法 为所有新增映射条目添加类型检查逻辑
        set(key, value) {
            if (this.keyType && typeof key !== this.keyType) {
                throw new TypeError(`${key} if not of type ${this.keyType}`);
            }
            if (this.valueType && typeof value !== this.valueType) {
                throw new TypeError(`${value} is not of type ${this.valueType}`);
            }
            return super.set(key, value);
        }
    }
}
```

在构造函数中使用`super()`，有几个重要的规则需要知道：

- 如果用`extends`定义了类，那么该类的构造函数(若有)必须用`super()`调用父类的构造函数。
- 如果没有在子类中定义构造函数，解释器会自动创建。该隐式定义的构造函数会取得传给它的值，然后将该值传给`super()`。
- 通过`super()`调用父类构造函数前，不能在构造函数中使用`this`关键字。
- 没有使用`new`关键字调用的函数中，特殊表达式`new.target`的值是`undefined`。在构造函数中，`new.target`引用的是被调用的构造函数。当子类构造函数被调用并使用`super()`调用父类构造函数时，该父类构造函数通过`new.target`可以获取子类构造函数。父类可以用`new.target.name`来记录日志消息。

覆盖父类方法的方法可以在覆盖方法的开头、中间或末尾调用。

### 委托而不是继承

面向对象编程领域奉行的一个准则：开发者应该"能组合就不继承"(favor composition over inheritance)。

也就是在类中创建另一个类的实例，并在需要时委托该实例去做我们希望的事。这样只需要包装或组合其它类即可。这种委托策略常称为"组合"(composition)。

### 类层次与抽象类

可能日常中更喜欢使用组合，因此几乎不会用到`extends`，除非用某个库或框架，要求必须扩展其基类。



