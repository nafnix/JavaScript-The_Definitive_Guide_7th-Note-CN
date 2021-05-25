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

