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

可迭代对象与迭代器的一个重要特点：若计算下个值需要一定的计算量，则相应计算会推迟到实际需要下个值时再发生。

对字符串中的单词进行懒惰迭代：

```js
function words(s) {
    var r = /\s+|$/g;						// 匹配一个或多个空格或末尾
    r.lastIndex = s.match(/[^ ]/).index;	// 开始匹配第一个非空格
    return {								// 返回一个可迭代的迭代器对象
        [Symbol.iterator]() {
            return this;
        },
        next(){
            let start = r.lastIndex;		// 从上次匹配结束的地方恢复
            if(start < s.length){			// 如果还没处理完
                let match = r.exec(s);		// 匹配下个单词边界
                if(match) {					// 如果找到一个单词就返回它
                    return { value: s.substring(start, match.index) };
                }
            }
            return {done: true};		// 否则返回表示处理完成的结果
        }
    };
}

[...words(" abc def  ghi! ")]		// ["abc", "def", "ghi!"]
```

### "关闭"迭代器：return()方法

有时迭代器不一定会跑完，例如`for/of`可能遇到`break`、`return`或异常终止。

如果迭代在`next()`返回`done`属性为`true`的迭代结果前停止，那么解释器就会检查迭代器是否有`return()`方法。若有，解释器就调用，让迭代其有机会关闭文件、释放内存，或做一些其他工作。

`return()`必须返回对象，返回非对象会报错。

## 生成器

是种使用新ES6语法定义的迭代器，适合要迭代的值不是某个数据结构的元素，而是计算结果的场景。

调用生成器函数不会实际执行函数体，而是返回一个生成器对象。该对象是个迭代器。调用其`next()`会导致生成器函数的函数体从头(或当前位置)开始执行，直至遇见一个`yield`语句。`yield`是ES6的新特性，类似`return`语句。

```js
// 该生成器函数回送一组素数(10进制)
// 调用该函数不会运行下述代码 只会返回一个生成器对象 调用该对象的next()会开始运行 直至一个yield语句为next()方法提供返回值
function* oneDigitPrimes(){	
    yield 2;
    yield 3;
    yield 5;
    yield 7;
}

// 得到一个生成器
let primes = oneDigitPrimes();

// 生成器是个迭代器对象 可以迭代回送的值
primes.next().value			// 2
primes.next().value			// 3
primes.next().value			// 5
primes.next().value			// 7
primes.next().done			// true

// 生成器有个Symbol.iterator方法 也是可迭代对象
primes[Symbol.iterator]()	// primes

// 可以像其他可迭代对象一个用生成器
[...oneDigitPrimes()]		// [2,3,5,7]

let sum = 0
for(let prime of oneDigitPrimes()) sum += prime;
sum							// 17
```

也可以用表达式定义生成器：

```js
const seq = function*(from, to) {
    for(let i = from; i <= to; i++) yield i;
};
[...seq(3, 5)]		// [3,4,5]
```

类和对象中可以简写：

```js
let o = {
    x: 1, y: 2, z: 3,
    *g() {
        for (let key of Object.keys(this)) {
            yield key;
        }
    }
};

[...o.g()]		// ["x", "y", "z", "g"]
```

不能用箭头函数语法定义生成器函数

### 生成器的示例

若确实生成自己通过某种计算回送的值，生成器还有更大用处。例如回送斐波那契数：

```js
// 无限循环 若通过...使用会一直循环到内存耗尽
function* fibonacciSequence() {
    let x = 0, y = 1;
    for(;;) {
        yield y;
        [x, y] = [y, x+y]		// 解构赋值
    }
}

// 返回第n个斐波那契数
function fibonacci(n) {
    for(let f of fibonacciSequence()) {
        if (n-- <= 0) return f;
    }
}
fibonacci(20)			// 10946
```

配合下述`take()`生成器，无穷生成器可以派上更大用场：

```js
// 回送指定可迭代对象的前n个元素
function* take(n, iterable) {
    let it = iterable[Symbol.iterator]();		// 取得可迭代对象的生成器
    while(n-- > 0) {				// 循环n次
        let next = it.next()		// 从迭代器中取得下一项
        if(next.done) return;		// 若没有更多值 直接返回
        else yield next.value;		// 否则回送该值
    }
}

// 包含前5个斐波那契数的数组
[...take(5, fibonacciSequence())]		// [1,1,2,3,5]
```

交替回送多个可迭代对象的元素：

```js
// 拿到一个可迭代对象的数组 交替回送它们的元素
function* zip(...iterables) {
    // 取得每个可迭代对象的迭代器
    let iterators = iterables.map(i => i[Symbol.iterator]());
    let index = 0;
    while(iterators.length > 0) {				// 在还有迭代器的情况下
        if(index >= iterators.length){			// 如果到了最后一个迭代器
            index = 0;							// 返回至第一个迭代器
        }
        
        let item = iterators[index].next();		// 从下个迭代器中取得下一项
        if(item.done) {							// 如果该迭代器完成
            iterators.splice(index, 1);			// 则从数组中删除它
        }else {									// 否则
            yield item.value;					// 回送迭代的值
            index++;							// 并前进到下个迭代器
        }
    }
}

// 交替3个可迭代对象
[...zip(oneDigitPrimes(), "ab", [0])]			// [2, "a", 0, 3, "b", 5, 7]
```

### yield*与递归生成器

也可以按顺序回送它们的元素的生成器函数：

```js
function* sequence(...iterables) {
    for(let iterable of iterables) {
        for(let item of iterable) {
            yield item;
        }
    }
}

[...sequence("abc", oneDigitPrimes())]		// ["a", "b", "c", 2, 3, 5, 7]
```

这种生成器函数中回送其他可迭代对象元素的操作很常见，所以ES6为它定义了特殊语法，使用`yield*`：

```js
function* sequence(...iterables) {
    for(let iterable of iterables) {
        yield* iterable;
    }
}

[...sequence("abc", oneDigitPrimes())]		// ["a", "b", "c", 2, 3, 5, 7]
```

`yield`和`yield*`只能在生成器函数中使用，不能在箭头函数中。

可以用`yield*`定义递归生成器，利用该特性可以通过简单的非递归迭代遍历递归定义的树结构。

## 高级生成器特性

### 生成器函数的返回值

生成器函数也可以返回值。

手工迭代时可以通过显式调用`next()`得到：

```js
function* oneAndDone(){
    yield 1;
    return "done";
}

// 正常迭代中不会出现返回的值
[...oneAndDone()]			// [1]

// 显式调用next()可以得到
let generator = oneAndDone();
generator.next()			// {value: 1, done: false}
generator.next()			// {value: "done", done: true}

// 如果生成器已经完成 则不会再有返回值
generator.next()			// {value: undefined, done: true}
```

### yield表达式的值

前面将`yield`看成是一个产生值但没有自己的值的语句。事实上，`yield`是个表达式(回送表达式)，可以有值。

`yield`关键字后的表达式会被求值，该值成为`next()`调用的返回值。此时，生成器函数就在求值`yield`表达式的中途停了下来。下次调用生成器的`next()`方法时，传给`next()`的参数回变成暂停的`yield`表达式的值。生成器通过`yield`向调用者返回值，而调用者通过`next()`向生成器传值。生成器和调用者是两个独立的执行流，它们交替传值(和控制权)：

```js
function* smallNumbers() {
    console.log("next()第一次被调用 参数被丢弃");
    let y1 = yield 1;		// y1 == "b"
    console.log("next()第二次被调用 参数是", y1);
    let y2 = yield 2;		// y2 == "c"
    console.log("next()第三次被调用 参数是", y2);
    let y3 = yield 3;		// y3 == "d"
    console.log("next()第四次被调用 参数是", y3);
    return 4;
}

let g = smallNumbers();
console.log("创建了生成器 代码未运行");
let n1 = g.next("a");			// n1.value == 1
console.log("生成器回送", n1.value);
let n2 = g.next("b");			// n2.value == 2
console.log("生成器回送", n2.value);
let n3 = g.next("c");			// n3.value == 3
console.log("生成器回送", n3.value);
let n4 = g.next("d");			// n4 == {value: 4, done: true}
console.log("生成器回送", n4.value);
```

返回：

```
创建了生成器 代码未运行
next()第一次被调用 参数被丢弃
生成器回送 1
next()第二次被调用 参数是 b
生成器回送 2
next()第三次被调用 参数是 c
生成器回送 3
next()第四次被调用 参数是 d
生成器回送 4
```

### 生成器的return()和throw()方法

可以调用生成器的`return()`和`throw()`改变生成器的控制流。调用它们会导致生成器返回值或抛出异常，就像生成器函数中的下一条语句是`return`或`throw`一样。

可以在生成器函数中用`try/finally`语句保证生成器返回时做些必要的清理工作。

生成器的`throw()`也为我们提供了(以异常形式)向生成器发送任意信号的途径。调用`throw()`方法会导致生成器函数抛出异常。

当生成器使用`yield*`回送其它可迭代对象时，调用生成器的`next()`方法会导致调用该可迭代对象`next()`方法，`return()`和`throw()`亦是如此。

### 关于生成器的最后几句话

生成器是强大的通用控制结构，赋予我们通过`yield`暂停计算并在未来某个时刻以任意输入值重新启动计算的能力。

可以使用生成器在单线程JS代码中创建某种协作线程系统。也可以利用生成器来掩盖程序中的异步逻辑。

