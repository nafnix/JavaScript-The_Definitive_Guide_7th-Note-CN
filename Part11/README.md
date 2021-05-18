# JavaScript标准库

## 集合与映射

### Set类

集合与数组的不同之处：

- 没有索引或顺序
- 不允许重复
- 值要么是集合成员要么不是
- 该值不可能在一个集合中出现多次

可以用`Set()`构造函数创建集合对象：

```js
let s = new Set();
let t = new Set([1, s]);
```

`Set()`构造函数的参数不一定是数组，但必须是个可迭代对象(包括其他集合)：

```js
let t = new Set(s);
let unique = new Set("Mississippi");		// {"M", "i", "s", "p"}
```

集合的`size`属性类似数组的`length`属性，保存着集合包含多个值：

```js
unique.size			// 4
```

可以通过`add()`、`delete()`、`clear()`方法给它添加元素或从中删除元素。

```js
let s = new Set();
s.size			// 0
s.add(1)		// 添加一个数值
s.size			// 1
s.add(1)		// 重复
s.size			// 1
s.add(true)		// 添加一个布尔值
s.size			// 2
s.add([1,2,3])	// 添加一个数组
s.delete(1)		// true 成功删除元素1
s.size			// 2
s.clear()		// 清空集合
s.size			// 0
```

- `add()`始终返回调用它的集合，可以`s.add().add().add()`。
- `delete()`返回一个布尔值，若指定的值是个集合成员，那就删除并返回`true`，否则什么也不做并返回`false`。

集合成员通过严格相等判断是否重复。

实践中，集合常用于检查某个值是不是集合的成员，为此要用`has`方法：

```js
let oneDigitPrimes = new Set([2, 3, 5, 7]);
oneDigitPrimes.has(2)		// true
oneDigitPrimes.has(3)		// true
oneDigitPrimes.has(4)		// false
oneDigitPrimes.has("5")		// false
```

集合专门为成员测试做了优化，无论有多少成员，`has()`方法都非常快。

数组的`includes`方法也执行成员测试，但其执行速度与数组大小成反比。因此，使用数组作为集合比使用真正的`Set`对象要慢得多。

`Set`可迭代：

```js
let sum = 0;
for(let p of oneDigitPrimes) {
    sum += p;
}

sum		// 17

[...oneDigitPrimes]				// [2,3,5,7]
Math.max(...oneDigitPrimes)		// 7
```

JS的`Set`类会记住元素的插入顺序，且始终按该顺序迭代集合：第一个元素第一个迭代，最后添加的元素最后迭代。

除了可以迭代，`Set`类也实现了一个`forEach`方法：

```js
let product = 1;
oneDigitPrimes.forEach(n => { product *= n; });
product			// 210 2*3*5*7
```

### Map类

可以使用`Map`创建映射对象：

```js
let m = new Map();			// 创建一个新的、空映射
let n = new Map([			// 初始化新映射 包含字符串到数值的映射
    ["one", 1],
    ["two", 2]
]);
```

`Map()`构造函数的可选参数是一个可迭代对象，产出值为包含两个元素的数组`[key, value]`。

也可以用`Map()`构造函数复制其他映射，或者从已有对象复制属性名和值：

```js
let copy = new Map(n);
let o = { x: 1, y: 2 };
let p = new Map(Object.entries(o));			// 相当于new Map([["x", 1], ["y", 2]])
```

- `get(键)`可以查询关联的值
- `set()`可以添加新的键/值对或修改已存在的键
- `has()`可以检查映射中是否包含指定键
- `remove()`可以删除指定键值对
- `clear()`可以删除所有键值对
- `size`可以查看有多少键

```js
let m = new Map()
m.size				// 0
m.set("one", 1);
m.set("two", 2);
m.size
m.get("two")		// 2
m.get("three")		// undefined
m.set("one", true);	// 修改与已有的键关联的值
m.size				// 2
m.has("one")		// true
m.has(true)			// false 没有键true
m.delete("one")		// true 键"one"存在且删除成功
m.size				// 1
m.delete("three")	// false 删除不存在的键失败
m.clear()			// 删除映射中所有的键和值
```

类似`Set`的`add()`，映射的`set()`方法可以连缀调用：

```js
let m = new Map().set("one", 1).set("two", 2).set("three", 3);
m.size			// 3
```

任何JS值都可以在映射中作为键或值，包括`null`、`undefined`和`NaN`，以及对象和数组等引用类型。

```js
let m = new Map();
m.set({}, 1);		// 映射空对象到值1
m.set({}, 2);		// 映射另一个空对象到值2
m.size				// 2 这个映射中有两个键
m.get({})			// undefined 但这个空对象不是映射的键
m.set(m, undefined);	// 把映射自身映射到值undefined
m.has(m)				// true
m.get(m)				// undefined
```

映射对象可以迭代，如果对映射对象使用扩展操作符，会得到一个数组的数组：

```js
let m = new Map([["x", 1], ["y", 2]]);
[...m]			// [["x", 1], ["y", 2]]

for(let [key, value] of m) {
    // 第一次迭代 key === "x", value === 1
    // 第二次迭代 key === "y", value === 2
}
```

如果只想迭代映射的键，可以用`keys()`，对于值可以用`values()`。

映射也有`forEach()`方法，通过这个最早由`Array`类实现的方法也可以迭代映射：

```js
m.forEach((value, key)) => {		// 是value, key 而非key, value
    // 第一次迭代 value === 1 key === "x"
    // 第二次迭代 value === 2 key === "y"
}
```

映射的`forEach()`方法先传映射的值，后传映射的键。

### WeakMap和WeakSet

WeakMap(弱映射)类是Map类的一个变体(不是子类)，它不会阻止键值被当作垃圾收集。

常规映射对自己的键值保持着"强"引用，即使对它们的所有其他引用都不存在了，仍然可以通过映射访问这些键。

`WeabMap()`构造函数与`Map()`的类似，但与映射有明显区别：

- `WeakMap`的键必须是对象或数组，原始值不受垃圾收集控制，不能作为键。
- `WeakMap`只实现了`get()`、`set()`、`has()`、`delete()`方法。
- `WeakMap`不是可迭代对象，所以没有定义`keys()`、`values()`、`forEach()`方法。
- `WeakMap`没有`size`属性，因为弱映射的大小可能随着对象被当作垃圾收集而随时改变。

`WeakMap`的主要用途是实现值与对象的关联而不导致内存泄漏。

`WeakSet`(弱集合)实现了一组对象，不会妨碍这些对象被作为垃圾收集。

`WeakSet()`构造函数与`Set()`构造函数类似，如同弱映射和映射一样，弱集合与集合也有着类似的区别。

- `WeakSet`不允许原始值作为成员
- `WeakSet`只实现了`add()`、`has()`、`delete()`方法，且不可迭代
- `WeakSet`没有`size`属性

主要应用场景类似`WeakMap`。

## 定型数组与二进制数据

ES6新增定型数组(typed array)，与这些语言的低级数组非常接近。

定型数组严格来讲并不是数组(`Array.isArray()`对它们返回`false`)。

- 定型数组的元素全部都是数值。
- 创建定型数组时必须指定长度，且该长度不能再改变。
- 定型数组的元素在创建时始终都会被初始化为`0`。

### 定型数组的类型

JS没定义`TypedArray`类，而是定义了11种定型数组，每种都有自己的元素类型和构造函数：

| 构造函数              | 数值类型                   |
| --------------------- | -------------------------- |
| `Int8Array()`         | 有符号字节                 |
| `Uint8Array()`        | 无符号字节                 |
| `Uint8ClampedArray()` | 无符号字节(上溢不归零)     |
| `Int16Array()`        | 有符号16位短整数           |
| `Uint16Array()`       | 无符号16位短整数           |
| `int32Array()`        | 有符号32位短整数           |
| `Uint32Array()`       | 无符号32位短整数           |
| `BigInt64Array()`     | 有符号64位BigInt值(ES2020) |
| `BigUint64Array()`    | 无符号64位BigInt值(ES2020) |
| `Float32Array()`      | 32位浮点值                 |
| `Float64Array()`      | 64位浮点值：常规JS数值     |

`Float64Array`的元素与常规JS数值是同一种类型。`Float32Array`的元素精度较低、表示的范围也更小，但只占用一半内存(这个类型对应C和Java中的`float`)。

`Uint8ClampedArray`是`Uint8Array`的一种特殊变体：

- `Uint8Array`：如果要存储到数组元素的值大于255或小于0，这个值会"翻转"为其他值。涉及计算机内存的底层工作机制，速度非常快。
- `Uint8ClampedArray`：会额外做些类型检查，若要存储的值大于255或小于0，那会"固定"为255或0，而不会翻转。

上面每种定型数组构造函数都有一个`BYTES_PER_ELEMENT`属性，根据类型不同，该属性值可能是1、2、4、8。

### 创建定型数组

传入一个表示数组元素个数的数值参数：

```js
let bytes = new Uint8Array(1024);		// 1024字节
let matrix = new Float64Array(9);		// 3×3矩阵
let point = new Int16Array(3);			// 3D空间中的一个点
let rgba = new Uint8ClampedArray(4);	// 4字节的RGBA像素值
let sudoku = new Int8Array(81);			// 9×9的数独网格
```

若以这种方式创建定型数组，则数组元素一定会全部初始化为`0`、`0n`或`0.0`。

可以在创建它们时指定值：

```js
let white = Uint8ClampedArray.of(255, 255, 255, 0);		// RGBA不透明白色
```

如果只是用带一个参数版本的`from()`，那么可以把`.from`去掉而直接把可迭代或类数组对象传给构造函数，结果完全相同。

构造函数和`from()`都支持复制已有的定型数组：

```js
let ints = Uint32Array.from(white);		// 同样4个数值 但变成整数
```

为适应类型限制，已有的值可能被截短，且不会有警告，也不会报错。

```js
Uint8Array.of(1.23, 2.99, 45000)		// new Uint8Array([1, 2, 200])
```

还有种创建定型数组的方式，该方式要用到`ArrayBuffer`类型。`ArrayBuffer`是对一块内存的不透明引用。只要传入想分配内存的字节数就行：

```js
let buffer = new ArrayBuffer(1024*1024);
buffer.byteLength		// 1024*1024 1M
```

`ArrayBuffer`类不允许读取或写入分配的任何字节。但是可以创建使用该缓冲区内存的定型数组，通过该数组来读取或写入该内存。此时定型数组构造函数的参数：

1. 第一个参数：`ArrayBuffer`
2. 第二个参数：该缓冲区内的字节偏移量，若省略，则数组会使用缓冲区的所有内存
3. 第三个参数：数组的长度，若省略，则数组使用从起点位置到缓冲区结束的所有可用内存

对于这种形式的定型数组，数组的内存必须是对齐的，所以如果指定了字节偏移量，那么该值应该是类型大小的倍数。例如，`Int32Array()`构造函数要求必须是4的倍数，而`Float64Array()`则要求必须是8的倍数。

示例：

```js
let asbytes = new Uint8Array(buffer);				// 按字节查看
let asints = new Int32Array(buffer);				// 按32位有符号整数查看
let lastK = new Uint8Array(buffer, 1023*1024);		// 按字节查看最后一千字节
let ints2 = new Int32Array(buffer, 1024, 256);		// 按256位整数查看第二个一千字节
```

所有定型数组底层都有个`ArrayBuffer`，即便没有明确指定。

所有定型数组都有`buffer`属性，引用自己底层的`ArrayBuffer`对象。之所以需要直接使用`ArrayBuffer`对象，是因为有时候可能需要一个缓冲区的多个定型数组视图。

### 使用定型数组

定型数组不是真正的数组，但重新实现了多数数组方法，所以几乎可以像使用数组一样使用它们：

```js
let ints = new Int16Array(10);
ints.fill(3)
```

### 定型数组的方法与属性

`set()`方法用于一次性设置定型数组中的多个元素，也就是把其他常规数组或定型数组的元素复制到当前定型数组中：

```js
let bytes = new Uint8Array(1024);			// 1K缓冲区
let pattern = new Uint8Array([0, 1, 2, 3]);	// 4字节的数组
bytes.set(pattern);			// 把它们复制到另一个字节数组的开头
bytes.set(pattern, 4);		// 使用不同的偏移量再复制一次
bytes.set([0,1,2,3], 8);	// 或直接从一个常规数组复制值
bytes.slice(0, 12);			// new Uint8Array([0, 1, 2, 3, 0, 1, 2, 3, 0, 1, 2, 3])
```

定型数组也有一个`subarray`方法，返回调用它的定型数组的一部分：

```js
let ints = new Int16Array([0,1,2,3,4,5,6,7,8,9]);			// 10个短整数
let last3 = ints.subarray(ints.length-3, ints.length);		// 其中最后3个
last3[0]			// 7 等同于ints[7]
```

`subarray`与`slice`的区别：

- `slice`以新的、独立的定型数组返回指定的元素，不与原始数组共享内存
- `subarray()`则不复制内存，只返回相同底层值的一个新视图

每个定型数组都有三个属性与底层的缓冲区相关：

```js
last3.buffer 					// 定性数组的ArrayBuffer对象
last3.buffer === ints.buffer	// true 同一个缓冲区的视图

last3.byteOffset				// 14 这个视图从缓冲区的字节14开始

last3.byteLength				// 6 这个视图长度为6字节(3个16位整数长)

last3.buffer.byteLength			// 20 底层缓冲区长度为20字节
```

- `buffer`属性是数组的`ArrayBuffer`
- `byteOffset`是数组数据在这个底层缓冲区的起点位置
- `byteLength`是数组数据的字节长度

对于任何定型数组`a`，以下不变式都成立：

```js
a.length * a.BYTES_PER_ELEMENT === a.byteLength		// true
```

`ArrayBuffer`是不透明的字节块。通过定型数组可以访问它里面的字节，但是`ArrayBuffer`本身不是定型数组，可以对`ArrayBuffer`使用数值索引，但是并不能访问缓冲区的字节，只会导致难解的bug。

还可以通过创建一个初始化的定型数组，然后再用该数组的缓冲区创建其他视图：

```js
let bytes = new Uint8Array(1024);				// 1024字节
let ints = new Uint32Array(bytes.buffer);		// 或者256个整数
let floats = new Float64Array(bytes.buffer);	// 或者128个双精度浮点数
```

### DateView与字节序

使用定型数组可以查看相同字节序列的8、16、32或64位视图。

字节序：多个字节排列为更长机器字的顺序。

为效率考虑，定型数组使用底层硬件的原生字节序：

- 在小端系统中，`ArrayBuffer`中的字节排列顺序为低字节到高字节
- 在大端系统中，字节排列顺序为高字节到低字节

可以用以下代码确定底层平台的字节序：

```js
let littleEndian = new Int8Array(new Int32Array([1]).buffer)[0] === 1
```

`DateView`类可以显式指定读、写`ArrayBuffer`值时的字节序：

```js
// 假设要处理一个二进制字节的定型数组
// 首先 创建DataView对象 以便从字节中灵活地读取值
let view = new DataView(bytes.buffer, bytes.byteOffset, bytes.byteLength);

let int = view.getInt32(0);			// 从字节0开始按大端字节序读取有符号整数
int = view.getInt32(4, false);		// 下一个整数还是大端字节序
int = view.getUint32(8, true);		// 下一个整数是小端字节序且无符号

view.setUint32(8, int, false);		// 将其以大端字节序写回缓冲区
```

`DateView`为除了`Uint8ClampedArray`地定型数组类定义了`10`个`get`方法。这些方法地名字类似`getInt16`、`getUint32()`、`getBigInt64()`和`getFloat64()`。

1. 第一个参数：`ArrayBuffer`中的字节偏移量，表示读取值的开始位置。
2. 第二个参数：除了`getInt8`和`getUint8`之外，布尔值，`false`或空使用大端字节序，`true`使用小端字节序。

`DateView`也定义了10个对应的设置方法，用于向底层`ArrayBuffer`写入值。这些方法：

1. 第一个参数：偏移量，表示写入值的开始位置。
2. 第三个参数：除了`setInt8`和`setUint8`，布尔值，`false`或空使用大端字节序格式写入值，`true`使用小端字节序写入值(最低有效字节在前)

定型数组和`DateView`提供了处理二进制数据所需的全部工具，可以让我们编写能够解压ZIP文件或者从JPEG文件中提取元数据之类的JS程序。

## 正则表达式与模式匹配

正则表达式是一种描述文本模式的对象。

正则表达式语法事正则表达式自己的"迷你"编程语言。

### 定义正则表达式

声明正则表达式：

```js
// 创建了一个RegExp对象 并将它的值赋给pattern 匹配任意以字母"s"结尾的字符串
let pattern = /s$/;
let pattern = new RegExp("s$");
```

#### 字面量字符

JS正则表达式语法通过以反斜杠`\`开头的转义序列也支持一些非字母字符：

| 字符         | 匹配目标                                                     |
| ------------ | ------------------------------------------------------------ |
| 字母数字字符 | 自身                                                         |
| `\0`         | NUL字符(`\u000`)                                             |
| `\t`         | 制表符(`\u0009`)                                             |
| `\n`         | 换行符(`\000A`)                                              |
| `\v`         | 垂直制表符(`\u000B`)                                         |
| `\f`         | 进纸符(`\u000C`)                                             |
| `\r`         | 回车符(`\u000D`)                                             |
| `\xnn`       | 十六进制数值`nn`指定的拉丁字符。例如，`\x0A`等同于`\n`       |
| `\uxxxx`     | 十六进制数值`xxxx`指定的Unicode字符。例如，`\u0009`等于同`\t` |
| `\u{n}`      | 码点`n`指定的Unicode字符，其中`n`是介于`0`到`10FFFF`之间的`1`到`6`个十六进制数字。<br />这种语法仅在使用`u`标志的正则表达式中支持 |
| `\cX`        | 控制`^X`。例如，`\cJ`等价于换行符`\n`                        |

下列英文标点符号在正则表达式中有特殊含义：

```
^ $ . * + ? = ! : | \ / ( ) [ ] { }
```

如果想在正则表达式中包含这些标点符号的字面值，必须在这些字符前价格反斜杠`\`。例如匹配一个反斜杠的字符串：`/\\/`

因为字符串也使用反斜杠作为转义字符，所以任何反斜杠都要写两次。

#### 字符类

把个别字面值字符放到方括号中可以组合成字符类。

| 字符     | 匹配目标                                                     |
| -------- | ------------------------------------------------------------ |
| `[...]`  | 方括号中任意一个字符                                         |
| `[^...]` | 不在方括号中的任意一个字符                                   |
| `.`      | 除换行或其他Unicode行终止符之外的任意字符。如果`RegExp`使用`s`标志，则句点匹配任意字符，包括终止符 |
| `\w`     | 任意ASCII单词字符。等价于`[a-zA-Z0-9_]`                      |
| `\W`     | 任意非ASCII单词字符。                                        |
| `\s`     | 任意Unicode空白字符                                          |
| `\S`     | 任意非Unicode空白字符                                        |
| `\d`     | 任意ASCII数字字符。等价于`[0-9]`                             |
| `\D`     | 任意非ASCII数字字符。                                        |
| `[\b]`   | 推个字符字面值(特例)                                         |

要在正则表达式中表示一个退格字符的字面值，要使用只包含一个元素的字符类：`/[\b]/`。

ES2018中，如果正则表达式使用了`u`标志，则支持字符类`\p{...}`及其排除性形式`\P{...}`。这些字符类建立在Unicode标准定义的属性基础上，表示的字符集可能随着Unicode标准的发展而变化。

数字匹配`\d`：

- 默认：匹配ASCII数字
- 匹配世界书写体系中任意十进制数字：`/\p{Decimal_Number}/u`
- 匹配任意语言非十进制数字：`/P{Decimal_Number}/u`(大写`P`)
- 匹配任意类数值字符(分数和罗马数字)：`/p{Number}`

`Decimal_Number`和`Number`非JS或正则特有，是Unicode标准定义的字符类别的名字。

字符匹配`\w`：

- 默认：匹配ASCII文本
- 模拟国际化版本：使用`\p`，`/[\p{Alphabetic}\p{Decimal_Number}\p{Mark}]/u`
- 完全兼容世界上各式各样的语言，还要添加`Connector_Punctuation`和`Join_Control`

`p`语言也支持定义匹配特定字母表或文字(`script`)中字符的正则表达式：

```js
let greekLetter = /\p{Script=Greek}/u;
let cyrillicLetter = /\p{Script=Cyrillic}/u;
```

#### 重复

也称贪婪匹配。

| 字符    | 含义                  |
| ------- | --------------------- |
| `{n,m}` | 匹配：$n\geq前项<m$   |
| `{n,}`  | 匹配前项 $n$ 或更多次 |
| `{n}`   | 匹配前项 $n$ 次       |
| `?`     | 匹配前项零次或一次    |
| `+`     | 匹配前项一次或多次    |
| `*`     | 匹配前项零次或多次    |

示例：

```js
let r = /d{2, 4}
r = /\w{3}\d?/		// 匹配3个字母后跟1个可选的数字
r = /\s+java\s+/	// 匹配"java"且前后跟一个或多个空格
r = /[^(]*/			// 匹配零个或多个非开始圆括号字符
```

#### 非贪婪重复

在贪婪匹配后加个问号，就可以指定非贪婪地重复，如`??`和`+?`和`*?`和`{1, 5}?`。

示例：

- `/a+/`：匹配一个或多个字符`a`，应用到字符串`aaa`时候，会匹配三个字母`aaa`。
- `/a+?/`：匹配一个或多个字符`a`，但是应用到字符串`aaa`时候，只会匹配第一个字母`a`。

但也不一定能得到期待的结果，例如匹配`aaab`：

- `/a+b/`：匹配到`aaab`
- `/a+?b/`：匹配到`aaab`，本意是匹配第一个`a`和`b`。但事实上该模式会匹配整个字符串，与贪婪的版本一致。
  - 这是因为正则表达式的匹配会从字符串的第一个位置开始查找匹配项。因为在字符串一开始就找到了，所以从后续字母开始的更短的匹配项就不在考虑之中了。

#### 任选、分组和引用

- 任选：`|`，`/ab/cd/ef/`，从左到右，忽略右边
- 分组：`()`，将独立的模式分组为子表达式，从而让这些模式可以被`|`、`*`、`+`、`?`当作一个整体。例如
  - `/java(script)?/`，后跟可选`script`
  - `/(ab/cd)+|ef/`，匹配字符串`ef`，包含一个或多个字符串`ab`或`cd`

圆括号的另一个作用：在完整的模式中定义子模式。当正则表达式成功匹配到一个目标字符串后，可以从目标字符串中提取出与圆括号包含的子模式对应的部分的文本。

例如在一个正则表达式中回引子表达式：

```js
/([Jj]ava([Ss]cript)?\sis\s(fun\w*))/		// 匹配示例JavaScript is fun
```

匹配位于一对单或双引号间的一个或多个字符。但不要求开始和结尾的引号匹配：

```js
/['"][^'"]*['"]/
```

如果想要引号匹配，可以使用引用：

```js
/['"][^'"]*\1/
```

如果不想让圆括号分组的子表达式生成数字引用，可以不用圆括号分组，而是开头用`(?:`，结尾用`)`：

```js
/([Jj]ava(?:[Ss]cript)?\sis\s(fun\w*))/		// 匹配示例JavaScript is fun
```

| 字符      | 含义                              |
| --------- | --------------------------------- |
| `\|`      | 任选                              |
| `(...)`   | 分组                              |
| `(?:...)` | 仅分组，不记住分组匹配的字符      |
| `\n`      | 匹配与第`n`个分组匹配的相同字符。 |

ES2018新增了"命名捕获组"，即给分组命名，使用`(?<名字>匹配内容)`，示例：

```js
/(?<city>\w+) (?<state[A-Z]{2}) (?<zipcode>\d{5})(?<zip9>-\d{4})?/
```

回引示例：

```js
/(?<quote>['"])[^'"]*\k<quote>/
```

#### 指定匹配位置

`\b`表示匹配可以发生的合法位置。

对于`/\sJava\s/`的匹配结果会包含左右两侧空格，且对于只有`Java`的句子匹配不了，可以用匹配(锚定)词边界的`\b`：`\bJava\b`。相应地，组件`\B`锚定与非词边界匹配的位置。

`/\B[Ss]cript/`匹配`JavaScript`和`postscript`，但不匹配`script`或`Scripting`。

`(?=表达式)`：向前查找断言，其中字符必须存在，但不实际匹配：

```js
/[Jj]ava([Ss]cript)?(?=\:)/
```

该模式匹配：`JavaScript: The Definitive Guide`中的`JavaScript`，但不匹配`Java in a Nutshell`中的`Java`。因为它后面没有`:`。如果后面`?=`表达式成立，则返回前面的内容。

若将`?=`改成`?!`，就变成了否定式向前查找，表示必须不存在断言中指定的字符：

```js
/Java(?!Script)([A-Z]\w*)
```

上述正则匹配`Java`后跟一个大写字母及任意数量的ASCII单词字符，但`Java`后不能是`Script`。

| 字符         | 含义                 |
| ------------ | -------------------- |
| `^`          | 匹配字符串开头       |
| `$`          | 匹配字符串末尾       |
| `\b`         | 匹配单词边界         |
| `\B`         | 匹配非单词边界的位置 |
| `(?=表达式)` | 肯定式向前查找断言   |
| `(?!表达式)` | 否定式向前查找断言   |

ES2018支持了向后查找断言，用法类似向前查找断言。

- 肯定式向后查找断言：`(?<=...)`
- 否定式向后查找断言：`(?<!...)`

示例：

肯定式向后查找断言，匹配5位的美国邮编，但仅限于前面是两位字母的州简写的情况：

```js
/(?<= [A-Z]{2} )\d{5}/
```

否定式向后查找断言，匹配前面不带Unicode货币符号的数字字符串：

```js
/(?<![\p{Currency_Symbol}d.])\d+(\.\d+)?/u
```

#### 标志

每个正则都可以带标志，用于修改其行为。

标志在正则表达式字面量中放在第二个斜杠后面，或在使用`RegExp()`构造函数时要以字符串形式作为第二个参数。

`g`：表示该正则是"全局性的"，意味要找到字符串中包含的所有匹配项，而非只是找到第一个匹配项。该标志不改变模式匹配的方式。

`i`：表示模式匹配不区分大小写。

`m`：表示皮皮额应以"多行"模式进行。`^`和`$`锚点应既匹配字符串的开头和末尾，也匹配任何一行的开头和末尾。

`s`：类似`m`。默认`.`在正则匹配中会排除行终止符。而在使用`s`后，会包含行终止符。

`u`：代表Unicode，能让正则匹配完整的码点而非匹配16位值。是ES6新增。若无特殊原因，应对所有正则都使用该标志。

`y`：表示正则是"有粘性的"，应该在字符串开头匹配或在紧跟前一个匹配的第一个字符处匹配。在应用给只想查找一个匹配项的正则表达式时，该标志的作用类似给正则加上了`^`锚点，将其锚定到字符串开头。对于用在字符串中反复查找所有匹配项的正则，该标志比较有用。

以上标志可以任意组合。

### 模式匹配的字符串方法

#### search()

接受一个正则参数，返回第一个匹配项起点字符位置，如果没有则返回`-1`：

```js
"JavaScript".search(/script/ui)		// 4
```

若参数不是正则，则会先将该参数传给`RegExp()`构造函数。

该方法不支持全局搜索，所以如果有`g`标志，则`g`标志会被省略。

#### replace()

执行搜索替换操作。

参数：

1. 正则
2. 替换后的字符串
3. 函数

带`g`标志替换所有，否则只替换一次。

如果第一个参数是字符串而非正则，那么会按字面值搜索替换。

如果替换字符串中出现$后跟一个数字，那么会把这两个字符替换位指定子表达式匹配的文本。可以通过它将字符串中的引号替换为其它字符：

```js
let quote = /"([^"]*)"/g;

'He said "stop"'.replace(quote, '《$1》')		// 'He said 《said》'
```

若是命名捕获组，可以通过名字而非数字来引用匹配的文本：

```js
let quote = /"(?<quotedText>[^"]*)"/g;
'He said "stop"'.replace(quote, '《$<quoteText》')		// 'He said 《stop》'
```

第三个参数作为函数会被用于计算替换的值。该替换函数被调用时会接收几个参数：

- 匹配的文本
- 若`RegExp`有捕获组，则后面的几个参数分别是这些捕获组匹配的子字符串
- 在字符串找到匹配项的位置
- 调用`replace()`方法的整个字符串
- 若`RegExp`包含命名捕获组，替换函数还会收到一个参数，该参数是个对象，属性名是捕获组的名字，属性值是匹配的文本：

```js
// 使用替换函数将字符串中的十进制整数转为十六进制
let s = "15 times 15 is 225";
s.replace(/\d+/gu, n => parseInt(n).toString(16))
```

#### match()

接受一个正则参数，返回数组，包含匹配结果。没有匹配项则返回`null`。

若有`g`标志，返回数组会包含在字符串里找到的所有匹配项，否则只查找第一个。

```js
// 带g标
"7 plus 8 equals 15".match(/\d+/g);			// ["7", "8", "15"]

// 无g标
let url = /(\w+):\/\/([\w.]+)\/(\S*)/;
let text = "Visit my blog at http://www.example.com/~david";
let match = text.match(url);
let fullurl, protocol, host, path;
if (match !== null) {
    fullurl = match[0];		// http://www.example.com/~david
    protocol = match[1];	// http
    host = match[2];		// www.example.com
    path = match[3];		// ~david
}
```

此外`match`若非全局搜索，那么将会有些额外属性。

- `input`：引用调用`match()`的字符串
- `index`：匹配项在字符串中的起始位置

若正则包含命名捕获组，那么还有个`groups`属性，值是个对象，对象属性即命名捕获组。

```js
let url = /(?<protocol)\w+):\/\/(?<host>[\w.]+)\/(?<path>\S*)/;
let text = "Visit my blog at http://www.example.com/~david";
let match = text.match(url);
match[0];
match.input			// text 引用调用match()的字符串
match.index			// 17 匹配项在字符串中的起始位置
match.groups.protocol	// "http"
match.groups.host		// "www.example.com"
match.groups.path		// "~david"
```

除了`g`标志，还有`y`标志也会对`match`有影响。如果同时有`g`和`y`，那么`match`返回包含所有匹配字符串的数组，就跟只设置了`g`而没有`y`一样。但第一个匹配项必须始于字符串开头，每个后续的匹配项必须从前一个匹配项的后一个字符开始。

若只有`y`，则`match`会尝试找到第一个匹配项，且默认该匹配项被限制在字符串开头。但该默认起始位置是可以修改的，设置`RegExp`对象的`lastIndex`属性可以指定匹配开始的位置。

若找到了匹配项，`lastIndex`属性会自动更新为匹配项后第一个字符的位置。

```js
let vowel = /[aeiou]/y;		// 粘着元音匹配
"test".match(vowel)			// null "test"开头的字符不是元音字母
vowel.lastIndex = 1			// 指定不同的起始位
"test".match(vowel)[0]		// "e"
vowel.lastIndex				// 2 自动更新
"test".match(vowel)			// null 位置2不是元音字母
vowel.lastIndex				// 0 匹配失败后会重置
```

给字符串的`match`方法传一个非全局正则表达式，相当于把字符串传给正则的`exec()`方法。这两种情况下返回的数组及其属性都相同。

#### matchAll()

ES2020中定义。

接受一个带`g`标志的正则。

返回一个迭代器，每次迭代都产生一个匹配对象。

```js
// 位于词边界间的一个或多个Unicode字母字符
const words = /\b\p{Alphabetic}+\b/gu;
const text = "This is a naive test of the matchAll() method.";
for(let word of text.matchAll(words)) {
    console.log(`Found '${word[0]}' at index ${word.index}`);
}
```

设置`matchAll`的`lastIndex`属性也可以告诉`matchAll`从字符串的哪个索引开始匹配。

但`matchAll`不会修改传入`RegExp`的`lastIndex`属性。

#### split()

该方法使用传入的参数作为分隔符，将调用它的字符串拆分为子字符串保存到一个数组里。

```js
"123,456,789".split(",")			// ["123", "456", "789"]
```

也接受正则参数：

```js
"1, 2, 3,\n4, 5".split(/\s*,\s*/)		// ["1", "2", "3", "4", "5"]
```

若调用`split()`时传入RegExp作为分隔符，且该正则包含命名捕获组，则捕获组匹配的文本也会包含在返回的数组中：

```js
const htmlTag = /<([^>]+)>/			// <后跟一个或多个非>字符再后跟>
"Testing<br/>1,2,3".split(htmlTag)	// ["Testing", "br/", "1,2,3"]
```

