# 迭代器与生成器

可迭代对象及其相关的迭代器是ES6的一个特性。

## 迭代器原理

可迭代对象：具有专有迭代器方法，且该方法返回迭代器对象的对象。

迭代器对象：具有`next()`方法，且该方法返回迭代结果对象的对象。

迭代结果对象：具有属性`value`和`done`的对象。

迭代一个可迭代对象：

1. 调用器迭代器方法获得迭代器对象
2. 重复调用该迭代器对象的`next()`方法，直至返回`done`属性为`true`的迭代结果对象。

可迭代对象的迭代器方法使用符号`Symbol.iterator`作为名字。可迭代对象`iterable`的简单`for/of`也可写作如下复杂形式：

```js
let iterable = [99];
let iterator = iterable[Symbol.iterator]();
for(let result = iterator.next(); !result.done; result = iterator.next()) {
    console.log(result.value)		// result.value 99
}
```

内置可迭代数据类型的迭代器对象本身亦可迭代：

```js
let list = [1,2,3,4,5];
let iter = list[Symbol.iterator]();
let head = iter.next().value;		// head 1
let tail = [...iter]				// tail [2,3,4,5]
```

## 实现可迭代对象

只要数据类型表示某种可迭代的结构，就应该将它们实现为可迭代对象。

示例：

```js
/*
 * Range对象表示一个数值范围{x:from <= x <= to}
 * Range定义了has()方法用于测试给定数值是否是该范围成员
 * Range是可迭代的 迭代其范围内的所有整数
 */

class Range {
    constructor(from, to) {
        this.from = from;
        this.to = to;
    }
    
    // 让Range对象像数值的集合一样
    has(x) { return typeof x === "number" && this.from <= x && x <= this.to; }
    
    // 使用集合表示法返回当前的字符串表示
    toString() { return `{ x | ${this.from} <= x <= ${this.to} }`; }
    
    // 通过返回一个迭代器对象 让Range对象可迭代
    // 该方法的名字是个特殊符号
    [Symbol.iterator]() {
        // 每个迭代器实例必须相互独立、互不影响地迭代自己的范围
        // 因此需要一个状态变量跟踪迭代的位置 从第一个大于等于from的整数开始
        let next = Math.ceil(this.from);		// 下一个返回的值
        let last = this.to;						// 不会返回大于它的值
        return {								// 迭代器对象
            // 该next()方法是迭代器对象的标志
            // 必须返回一个迭代器结果对象
            next() {
                return (next <= last) ? { value : next++ } : { done: true };
            },
            // 为方便起见 让迭代器本身也可迭代
            [Symbol.iterator](){ return this; }
        }
    }
}
```

使用：

```js
for(let x of new Range(1, 10)) console.log(x);		// 打印1到10
[...new Range(-2, 2)]
```

也可以定义返回可迭代值的函数。

下述两个函数可以代替`map`和`filter`：

```js
// 返回一个可迭代对象 迭代结果是对传入的可迭代对象的每个值应用f()的结果
function map(iterable, f) {
    let iterator = iterable[Symbol.iterator]();
    return {			// 该对象既是迭代器对象也是可迭代对象
        [Symbol.iterator]() { return this; },
        next() {
            let v = iterator.next();
            if(v.done) {
                return v;
            } else {
                return { value: f(v.value) };
            }
        }
    };
}

// 将一个范围内的整数映射为它们的平方并转换为一个整数
[...map(new Range(1, 4), x => x*x)]		// [1, 4, 9, 16]

// 返回一个可迭代对象 只迭代predicate返回true的函数
function filter(iterable, predicate) {
    let iterator = iterable[Symbol.iterator]();
    return {		// 该对象既是迭代器对象也是可迭代对象
        [Symbol.iterator]() { return this; },
        next() {
            for(;;) {
                let v = iterator.next();
                if(v.done || predicate(v.value)) {
                    return v;
                }
            }
        }
    }
}

// 筛选整数范围 只保留偶数
[...filter(new Range(1, 10), x => x % 2 === 0)]		// [2,4,6,8,10]
```

