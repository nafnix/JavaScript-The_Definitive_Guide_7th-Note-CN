# 数组

数组是值的有序集合，其中的值叫做元素，每个元素有一个数值表示的位置，叫做索引。

JS数组无类型限制，数组中的元素可以是任意类型，同一数组的不同元素可以是不同的类型。

JS数组是基于零且使用32位数值索引的，第一个元素的索引为0，最大的索引值是4_294_967_294，也就是数组最大包含4_294_967_295个元素。

JS数组是动态的，它们会按需增大或减少，所以不需要在创建数组时声明大小。

JS数组可以是稀疏的，元素不一定具有连续索引。`length`属性对于稀疏数组，`length`大于所有元素的最高索引。

数组从`Array.prototype`继承属性，该原型上定义了很多数组操作方法。

ES6新增了一批新的数组类，统称为"定型数组"(typed array)。定型数组具有固定长度和固定的数值元素类型。定型数组具有极高的性能，支持对二进制数据的字节级访问。

## 创建数组

### 数组字面量

```js
let empty = [];						// 空数组
let primes = [1,2,3,4,5]			// 5个数值元素
let misc = [1.1, true, "a", ]		// 3个不同类型的元素

let base = 1024;
let table = [base, base+1, base+2, base+3];

let b = [[1, {x: 1, y: 2}], [2, {x: 3, y: 4}]];

let count = [1, ,3];			// 索引1没有元素
let undefs = [,,];				// 该数组没有元素但长度为2
```

### 扩展操作符

ES6后，可以用扩展操作符`...`在另一个数组字面量里包含另一个数组的元素：

```js
let a = [1, 2, 3];
let b = [0, ...a, 4];			// b == [0, 1, 2, 3, 4]
```

扩展操作符是创建数组(浅)副本的一种便捷方式：

```js
let original = [1, 2, 3];
let copy = [...original];
copy[0] = 0;			// 修改copy不会影响original
original[0]				// 1
```

扩展操作符适用于任何可迭代对象：

```js
let digits = [..."0123456789ABCDEF"];
digits			// ["0", "1", "2", "3", "4", "5", "6", "7", "8", "9", "A", "B", "C", "D", "E", "F"]
```

集合对象是可迭代的，所以要去除数组中的重复元素，一种方式是把数组转为集合，然后再用扩展操作符把这个集合转换回数组：

```js
let letters = [..."hello world"];
[...new Set(letters)]			// ["h", "e", "l", "o", " ", "w", "o", "r", "l", "d"]
```

### Array()构造函数

```js
let a = new Array();			// 创建空数组
let a = new Array(10);			// 指定长度创建数组
let a = new Array(5, 4, 3, 2, 1, "testing, testing");
```

### Array.of()

ES6后，可以用`Array.of`来创建只包含一个数值元素的数组：

```js
Array.of()			// []
Array.of(10)		// [10]
Array.of(1, 2, 3)	/ [1, 2, 3]
```

### Array.from()

ES6新增。

期待一个可迭代对象或类数组对象作为第一个参数，并返回包含该对象的新数组。

若传入可迭代对象，则与使用扩展操作符效果一致。因此它也是创建数组副本的一种简单方式：

```js
let copy = Array.from(original);
```

`Array.form`也接受可选的第二个参数。若给第二个参数引入了一个函数，那么在构建新数组时，源对象的每个元素都会传入这个函数，该函数的返回值将代替原始值成为新数组的元素。

## 读写数组元素

```js
let a = ["world"];
let value = a[0]
a[1] = 3.14
let i = 2
a[i] = 3
a[i + 1] = "hello"
a[a[i]] = a[0]
```

JS会将数值索引转为字符串，也就是索引`1`会变成字符串`"1"`，然后再将这个字符串作为属性名。

可以使用负数或非整数作为索引数组。这时候数值会转成字符串，而字符串会作为属性名。因为该名字是非负整数，所以会被当作常规的对象属性，而非数组索引。

```js
a[-1.23] = true			// 创建属性"-1.23"
a["1000"] = 0			// 数组中第1001个元素
a[1.000] = 1			// 数组索引1 相当于a[1] = 1
```

因为数组索引其实是一种特殊的对象属性，所以JS里的数组没有"越界"错误。查询不存在的属性不会导致错误，而是返回`undefined`。

## 稀疏数组

稀疏数组就是其元素没有从0开始的索引的数组。正常情况下，数组的`length`属性表明数组中元素的个数。

若数组是稀疏的，则`length`属性的值会大于元素个数。可以用`Array()`构造函数创建稀疏数组，或直接给大于当前数组`length`的数组索引赋值：

```js
let a = new Array(5)			// 没有元素 但a.length是5
a = []							// 创建空数组 此时length为0
a[1000] = 0						// 赋值增加了一个元素 但length变成了1001
```

足够稀疏的数组通常是以比稠密数组慢、但内存占用少的方式实现的，查询这种数组的元素与查询常规对象属性的时间相当。

```js
let a1 = [,]			// length为1 没有元素
let a2 = [undefined]	// 有一个undefined元素
0 in a1 				// false 索引0没有元素
0 in a2					// true 索引0有个undefined元素
```

## 数组长度

```js
[].length
```

 若将`length`属性设为一个小于当前元素数量的数值，则大于该数值的元素项数都会被删除。

```js
a = [1,2,3,4,5]
a.length = 3
a.length = 0
a.length = 5
```

## 添加和删除数组元素

```js
let a = []
a[0] = "zero"
a[1] = "one"
```

也可以用`push()`方法在数组末尾添加一个或多个元素：

```js
let a = []
a.push("zero")
a.push("one", "two")
```

可以理解成`a[a.length] = value`，与之对应的是`pop()`，他将数组最后一个元素弹出。

删除元素：

```js
let a = [1, 2, 3]
delete a[2]
2 in a			// false
a.length		// 3 删除元素不影响数组长度
```

删除元素类似于(不完全等于)给该元素赋予`undefined`值。

## 迭代数组

```js
let letters = [..."Hello world"]
let string = "";
for (let letter of letters){
    string += letter
}
string 		// "Hello world"
```

若要对数组使用`for/of`循环，并且想知道每个数组元素的索引，可以用数组的`entries()`方法和解构赋值：

```js
let everyother = "";
for(let [index, letter] of letters.entries()) {
    if (index % 2 === 0) everyother += letter;			// 偶数索引的字母
}
```

另一种迭代数组的推荐方式是用`forEach()`。它是新的`for`循环，而是数组提供的一种用于自身迭代的函数式方式。要给`forEach()`传一个函数，然后`forEach()`会用数组的每个元素调用一次该函数：

```js
let uppercase = "";
letters.forEach(letter => {		// 箭头函数
    uppercase += letter.toUpperCase();
});
uppercase			// "HELLO WORLD"
```

不同于`for/of`循环，`forEach()`能够感知稀疏数组，不会对没有的元素数组调用函数。

也有其他的方式：

```js
// 将数组长度保存到局部变量里
for (let i = 0, len = letters.length; i < len; i++) {
    // 循环体不变
}

// 从后向前迭代数组
for (let i = letters.length - 1; i >= 0; i--) {
    // 循环体不变
}
```

上面两个都假定数组是稠密的。若不是这种情况，就应该在使用每个元素前对其测试：

```js
for (let i = 0; i < a.length; i++) {
    if( a[i] === undefined) continue;		// 跳过未定义及不存在的元素
}
```

## 多维数组

```js
let table = new Array(10);
for (let i = 0; i < table.length; i++) {
    table[i] = new Array(10);
}

for (let row = 0; row < table.length; row++) {
    for (let col = 0; col < table[row].length; col++) {
        table[row][col] = row*col;
    }
}

table[5][7]		// 35
```

## 数组方法

### 数组迭代器方法

所有这些方法都接受一个函数作为第一个参数，并对每个数组中元素都调用一次该函数。若数组是稀疏的，则不会对不存在的数组元素调用该函数。

#### forEach()

该方法迭代数组的每个元素，并对每个元素都调用以此我们指定的函数。

```js
let data = [1, 2, 3, 4, 5], sum = 0;
data.forEach(value => {sum += value;});					// sum == 15

data.forEach(function(v, i, a) { a[i] = v + 1; });		// data == [2,3,4,5,6]
```

#### map()

该方法调用其数组的每个元素分别传给我们指定的函数，返回该函数的返回值构成的数组：

```js
let a = [1, 2, 3]
a.map(x => x*x)			// [1, 4, 9]
```

#### filter()

该方法返回一个数组，该数组包含调用它的数组的子数组。传给该数组的函数应该是个断言函数，即返回`true`或`false`。

若函数返回`true`则传给该函数的元素就是`filter()`最终返回的子数组的成员：

```js
let a = [5, 4, 3, 2, 1]
a.filter(x => x < 3)			// [2, 1]
a.filter(x, i)					// [5, 3, 1]			
```

`filter`会跳过稀疏数组中缺失的元素，返回的数组始终是稠密的。所以可以用`filter()`方法像下面这样清理掉稀疏数组中的空隙：

```js
let dense = sparse.filter(() => true);
```

若想清理空隙的同时又想删除`undefined`和`null`元素：

```js
a = a.filter(x => x != undefined && x !== null);
```

#### find()与findIndex()

这两个类似`filter`，表现在它们都遍历数组，寻找断言函数返回真值的元素。不过它们会在找到第一个真值时停止迭代。

- `find`：
  - 找到：返回匹配的元素
  - 没找到：返回`undefined`
- `findIndex`：
  - 找到：返回匹配到的元素的索引
  - 没找到：`-1`

```js
let a = [1, 2, 3, 4, 5];
a.findIndex(x => x === 3)		// 2
a.findIndex(x => x < 0)			// -1
a.find(x => x % 5 === 0)		// 5
a.find(x => x % 7 === 0)		// undefined
```

#### every()与some()

这两个方法会对数组元素调用我们传入的断言函数，最后返回`true`或`false`。

`every`在数组所有的元素都满足断言函数时返回`true`。只要有一个不满足，就返回`false`

`some`在数组中只要有一个元素满足断言函数时返回`true`。只要有一个满足，就返回`true`

按照数学传统，对于空数组，`every`返回`true`，`some`返回`false`

#### reduce()和reduceRight()

这两个方法会用我们指定的函数归并数组元素，最终产生一个值。

```js
let a = [1, 2, 3, 4, 5]
a.reduce((x, y) => x+y, 0)			// 15
a.reduce((x, y) => x*y, 1)			// 120
a.reduce((x, y) => (x > y) ? x : y)	// 5
```

`reduce`的第一个参数是执行归并操作的函数。该归并函数的任务是把两个值归并或组合为一个值并返回该值。

第二个参数是可选的，是传给归并函数的初始值。

在不指定初始值的时候,`reduce`会使用数组的第一个元素作为初始值。

如果不指定初始值，那么在空数组上调用`reduce`会抛出`TypeError`。

`reduceRight`与`reduce`类似，不过是从高索引向低索引处理数组。

用例：

```js
// 计算 2^(3^4) 求幂具有从右到左的优先级
let a = [2, 3, 4]
a.reduceRight((acc, val) => Math.pow(val, acc))
```

`reduce`和`reduceRight`不是专门为数学计算而设计的。使用数组归并表达的算法很容易复杂话，导致难以理解。此时可能使用常规循环逻辑处理数组反倒更加容易阅读、编写和分析。

### 使用flat()和flatMap()打平数组

ES2019中，`flat()`方法用于创建并返回一个新数组：

```js
[1, [2, 3]].flat()			// [1, 2, 3]
[1, [2, [3]]].flat()		// [1, 2, [3]]
```

需要打平多层需要给`flat`传递一个数值参数：

```js
let a = [1, [2, [3, 4]]];
a.flat(1)			// [1, 2, [3, 4]]
a.flat(2)			// [1, 2, 3, [4]]
a.flat(3)			// [1, 2, 3, 4]
a.flat(4)			// [1, 2, 3 ,4]
```

`flatMap`类似`map`，不过返回的数组会被自动打平。调用`a.flatMap(f)`等同于(但效率远高于)`a.map(f).flat()`：

```js
let phrases = ["hello world", "the definitive guide"];
let words = phrases.flatMap(phrase => phrase.split(" "));
words			// ["hello", "world", "the", "definitive", "guide"];
```

`flatMap`允许把输入元素映射为空数组，这样打平后并不会有元素出现在输出数组里：

```js
// 将非负数映射为它们的平方根
[-2, -1, 1, 2].flatMap(x => x < 0 ? [] : Math.sqrt(x))		// [1, 2**0.5]
```

### 使用concat()添加数组

该方法创建并返回一个新数组：

```js
let a = [1, 2, 3]
a.concat(4, 5)				// [1, 2, 3, 4, 5]
a.concat([4, 5], [6, 7])	// [1, 2, 3, 4, 5, 6, 7]
a.concat(4, [5, [6, 7]])	// [1, 2, 3, 4, 5, [6, 7]]
a				// [1, 2, 3]
```

### 通过push()、pop()、shift()和unshift()实现栈和队列操作

使用`push`和`pop`可以使用JS数组实现先进后出的栈：

```js
let stack = []
stack.push(1, 2)			// [1, 2]
stack.pop()					// 2 stack==[1]
stack.push(3)				// [1, 3]
stack.pop()					// 3 stack==[1]
stack.push([4, 5])			// [1, [4, 5]]
stack.pop()					// [4, 5] stack==[1]
stack.pop()					// [1] stack==[]
```

- `unshift`用于在数组开头添加一个或多个元素，已有元素会像高位索引移动，并返回数组的新长度。
- `shift`删除并返回数组的第一个元素，所有元素会像低位索引移动

使用`unshift`和`shift`也能够实现栈，但是效率不如`push`和`pop`，因为每次在数组开头添加或删除元素都要向上或向下移动元素。

但也能用`push`和`shift`来实现：

```js
let q = []
q.push(1, 2)				// q==[1, 2]
q.shift()					// q==[2]		返回1
q.push(3)					// q==[2, 3]
q.shift()					// q==[3]		返回2
q.shift()					// q==[]		返回3
```

`unshift`如果有多个参数，那么会一次性插入数组。这意味这一次插入与多次插入后的数组顺序不一样：

```js
let a = []				// []
a.unshift(1)			// [1]
a.unshift(2)			// [2, 1]
a = []					// []
a.unshift(1, 2)			// [1, 2]
```

### 使用slice()、splice()、fill()和copyWithin()

#### slice()

该方法返回一个数组的切片(silce)或者子数组。该方法接受两个参数，分别用于指定要返回切片的起始位置。

```js
let a = [1, 2, 3, 4, 5]
a.slice(0, 3)			// [1, 2, 3]
a.slice(3)				// [4, 5]
a.slice(1, -1)			// [2, 3, 4]
a.slice(-3, -2)			// [3]
```

#### splice()

该方法是相对数组进行插入和删除的通用方法。`splice()`会修改调用它的数组。

第一个参数是插入或删除操作的七点。

第二个参数是要切割出来的元素个数。如果没有该参数，则从起点开始的所有元素被都删除。

`splice`返回被删除元素的数组，如果没有删除就返回空数组。

```js
let a = [1, 2, 3, 4, 5, 6, 7, 8];
a.splice(4)				// [5, 6, 7 ,8] a==[1,2,3,4]
a.splice(1,2)			// [2, 3] a==[1,4]
a.splice(1,1)			// [4] a==[1]
```

后面还能跟多个参数，表示要在第一个参数指定的位置插入到数组中的元素：

```js
let a = [1,2,3,4,5]
a.splice(2, 0, "a", "b")			// []			a == [1,2,"a","b",3,4,5]
a.splice(2, 2, [1, 2], 3)			// ["a", "b"]	a == [1,2,[1,2],3,3,4,5]
```

#### fill()

该方法将数组的元素或切片设置为指定的值。

修改调用的数组，也返回修改后的数组：

```js
let a = new Array(5)
a.fill(0)			// [0,0,0,0,0]
a.fill(9,1)			// [0,9,9,9,9]
a.fill(8,2,-1)		// [0,9,8,8,9]
```

第一个参数是要把元素设置的值

第二个参数是指定起始索引，是可选的。

第三个参数指定终止索引，到该索引为止(不包含该索引)，是可选的。

#### copyWithin()

该方法将数组切片复制到数组中的新位置。

修改数组并返回修改后的数组，但不会改变数组的长度。

第一个参数指定要把第一个元素复制到的目的的索引。

第二个参数指定要复制的第一个元素的索引，如果省略，则默认为`0`。

第三个参数指定要复制的元素切片的终止索引，如果省略，则使用数组长度。

```js
let a = [1,2,3,4,5]
a.copyWithin(1)				// [1,1,2,3,4]
a.copyWithin(2,3,5)			// [1,1,3,4,4]
a.copyWithin(0,-2)			// [4,4,3,4,4]
```

`copyWithin`本意是作为一个高性能方法，尤其对定型数组特别有用。模仿的是C的`memove()`函数。

### 数组索引与排序方法

#### indexOf()和lastIndexOf()

这两个方法从数组内搜索指定的值并返回第一个找到的元素的索引，如果没找到就返回`-1`。

`indexOf()`从前到后搜索数组，而`lastIndexOf()`从后向前搜索数组：

```js
let a = [0,1,2,1,0]
a.indexOf(1)			// 1	a[1]是1
a.lastIndexOf(1)		// 3	a[3]是1
a.indexOf(3)			// -1	没有元素的值是3
```

它们可以接受第二个参数，用于指定从哪个位置开始搜索。字符串也有这两个方法，区别在于第二个参数如果是负值就会被当作`0`。

#### includes()

ES2016的`includes()`接受一个参数，若数组有该值则返回`true`，否则返回`false`。

要注意的是：数组并非集合的有效表示方式，如果元素数量庞大，应该选择真正的`Set`对象。

`includes()`使用稍微不同的相等测试，认为`NaN`与自身相等：

```js
let a = [1, true, 3, NaN]
a.includes(true)			// true
a.includes(2)				// false
a.includes(NaN)				// true
a.indexOf(NaN)				// -1	无法找到
```

#### sort()

该方法对数组元素就地排序并返回排序后的数组。

不传参数时，表明按字母顺序对数组元素排序。

如果数组包含未定义的元素，它们会被排到数组末尾。

给其传一个比较函数作为参数，函数可以决定它的两个参数哪一个在排序后的数组中应该出现在前面：

1. 第一个参数应该出现在第二个参数前面，则比较函数应该返回一个小于`0`的数值
2. 第一个参数应该出现在第二个参数后面，则比较函数应该返回一个大于`0`的数值
3. 相等返回`0`

要对数组元素按照数值而非字母顺序排序：

```js
let a = [33, 4, 1111, 222];

a.sort()					// a == [1111, 222, 33, 4]

a.sort(function(a, b) {		// 传入一个比较函数
    return a-b;				// 取决于顺序 返回<0、0或>0
});							// a == [4, 33, 222, 1111] 数值顺序

a.sort((a, b) => b-a);		// a == [1111, 222, 33 ,4] 相反的数值顺序
```

若相对字符串数组做不区分大小写的字母排序，传入的比较函数应该将其两个参数都转为小写，然后再比较：

```js
let a = ["ant", "Bug", "cat", "Dog"];
a.sort()			// a == ["Bug", "Dog", "ant", "cat"] 区分大小写排序
a.sort(function(s,t) {
    let a = s.toLowerCase();
    let b = t.toLowerCase();
    if (a < b) return -1;
    if (a > b) return 1;
    return 0;
});

// a == ["ant", "Bug", "cat", "Dog"] 不区分大小写排序
```

#### reverse()

该方法反转数组元素的顺序，并返回反序后的数组。这是就地反序，直接修改原数组。

### 数组到字符串的转换

```js
let a = [1,2,3]
a.join()				// "1,2,3"
a.join(" ")				// "1 2 3"
a.join("")				// "123"
let b = new Array(10)	// 长度为10但没有元素的数组
b.join("-")				// "---------" 包含9个连字符的字符串
```

数组的`toString`方法的逻辑与没有参数的`join`一样。

### 静态数组函数

`Array.isArray()`用于确定一个未知数是不是数组：

```js
Array.isArray([])			// true
Array.isArray({})			// false
```

## 类数组对象

JS数组具有一些其他对象不具备的特殊特性：

- 数组的`length`属性会在新元素加入时自动更新
- 设置为`length`为更小的值会截断数组
- 数组从`Array.prototype`继承有用的方法
- `Array.isArray()`对数组返回`true`

事实上，只要对象有一个数值属性`length`，而且有相应的非负证书属性，那就完全可以视同为数组。

下述为一个常规对象添加属性，让其成为一个类数组对象，然后再遍历得到的伪数组的"元素"：

```js
let a = {}

let i = 0;
while(i < 10) {
    a[i] = i * i;
    i++;
}

a.length = i;

// 像遍历真正的数组一样遍历该对象
let total = 0;
for (let i = 0; j < a.length; j++) {
    total += a[j];
}
```

很多HTML文档的方法都是返回类数组对象。下述函数可以用于测试对象是否类数组对象：

```js
// 确定o是不是类数组对象
// 字符串和函数有数值length属性 但是通过typeof测试可以抛出
// 在客户端JS中DOM文本节点有数值length属性 可能需要加上o.nodeType !== 3测试来排除
function isArrayLike(o) {
    if (o && 							// o不是null、undefined等假值
        typeof o === "object" &&		// o是对象
        Number.isFinite(o.length) &&	// o.length是有限数值
        o.length >= 0 &&				// o.length是非负整数
        Number.isInteger(o.length) &&	// o.length是整数
        o.length < 4294967295 {			// o.length < 2^32 -1 
        	return true;		// 是类数组对象
        } else {
        	return false;		// 不是类数组对象
    	}
       )
}
```

多数JS数组方法有意地设计成了泛型方法，所以除了真正的数组，同样也可以用于类数组对象。但因为类数组对象不会继承`Array.prototype`，所以无法直接在它们上面调用数组方法。为此，可以用`Function.call()`方法来调用：

```js
let = {"0": "a", "1": "b", "2", "c", length: 3};		// 类数组对象
Array.prototype.join.call(a, "+")						// "a+b+c"
Array.prototype.map.call(a, x => x.toUpperCase())		// ["A", "B", "C"]
Array.prototype.slice.call(a, 0)						// ["a", "b", "c"] 真正的数组副本
Array.from(a)											// ["a", "b", "c"] 更容易地数组复制
```

## 作为数组的字符串

JS字符串地行为类似于UTF-16 Unicode字符地只读数组。除了用`charAt()`方法访问个别字符，也可以用方括号语法：

```js
let s = "test";
s.charAt(0)			// "t"
s[1]				// "e"
```

字符串与数组地行为类似意味我们可以对字符串使用泛型地字符串方法：

```js
Array.prototype.join.call("JavaScript", " ")		// "J a v a S c r i p t"
```

