# 第三章 类型、值和变量

## 概述与定义

1. 原始类型(不可修改)
   - 数值
   - 字符串
   - 布尔真值
   - null
   - undefined
   - ES6新增：Symbol(符号)
2. 对象类型：不是原始类型的值都是对象，属性的集合，可修改。

JS与静态语言更大的差别：函数和类不仅时语言的语法，本身就是可以被JS程序操作的值。

内存管理：JS解释器会执行自动垃圾收集。

## 数值

JS用IEEE 754标准定义的64位浮点格式表示数值。

### 整数字面量

- 十六进制：`0x`开头或`0X`开头，后跟十六进制数

ES6之后，支持二进制和八进制：

- 二进制：`0b`开头，例如`0b10101`
- 八进制：`0o`开头，例如`0o377`

### 浮点字面量

在实数后跟字母`e`或`E`，跟一个可选的加号或减号，再跟个整数指数。这种记数法表示的是实数值乘以`10`的指数次幂。

```js
3.14
2345.6789
.33333333333
6.02e2 				// 6.02 * 10²
1.4738223E+2		// 1.4738223 * 10²
```

可以用下划线将数值分隔：

```js
let billion = 1_000_000_000;
let bytes = 0*89_AD_CD_EF;
let bits = 0b0001_1101_0111;
let fraction = 0.123_456_789;
```

### JavaScript中的算术

上溢出时，结果时特殊的无穷值`Infinity`。

负数绝对值超出最大可表示负数的绝对值为`-Infinity`。

`0/0=NaN`

`NaN`与任何值比较都不想等，也不等于自己(`NaN`)。可以用`Number.isNaN(变量)`。

```js
Math.pow(2, 53)				// 2的53次方
Math.round(.6)				// 1.0 舍入到最接近的整数
Math.ceil(.6)				// 1.0 向上舍入到一个整数
Math.floor(.6)				// 0.0 向下舍入到一个整数
Math.abs(-5)				// 5 绝对值
Math.max(x, y, z)			// 返回最大的参数
Math.min(x, y, z)			// 返回最小的参数
Math.random()				// 伪随机数x 0 <= x < 1.0
Math.PI						// 圆周率
Math.E						// 自然对数的底数
Math.sqrt(3)				// 3**0.5 3的平方根
Math.pow(3, 1/3)			// 3**(1/3) 3的立方根
Math.sin(0)					// 三角函数 还有Math.cos和Math.atan等
Math.log(10)				// 10的自然对数
Math.log(100)/Math.LN10		// 以10为底100的对数
Math.log(512)/Math.LN2		// 以2为底512的对数
Math.exp(3)					// Math.E的立方
```

ES6新增：

```js
Math.cbrt(27)				// 3 立方根
Math.hypot(3, 4)			// 5 所有参数平方和的平方根
Math.log10(100)				// 2 以10为底的对数
Math.log2(1024)				// 10 以2为底的对数
Math.log1p(x)				// (1+x)的自然对数 精确到非常小的x
Math.expm1(x)				// Math.exp(x)-1 Math.log1p()的逆运算
Math.sign(x)				// 对<、==、>0的参数返回-1、0、1
Math.imul(2, 3)				// 6 优化的32位整数乘法
Math.clz32(0xf)				// 28 32位整数中前导0的位数
Math.trunc(3.9)				// 3 剪掉分数部分得到的整数
Math.fround(x)				// 舍入到最接近的32位浮点数
Math.sinh(x)				// 双曲线正弘 还有Math.cosh()和Math.tanh()
Math.asinh(x)				// 双曲线反正弘 还有Math.acosh()和Math.atanh()
```

### 二进制浮点数与舍入错误

通过JS操作实数的时候，数值表示通常是实际数值的近似值。

```js
let x = .3 - .2;
let y = .2 - .1;

x === y;		// false
x === .1;		// false
y === .1;		// false
```

可以考虑用等量整数。

### 通过BigInt表示任意精度整数

ES2020为JS定义了一种新的数值类型`BigInt`。

```js
1234n		// 一个不太大的BigInt字面量
0b111111n	// 二进制BigInt
0o7777n		// 八进制BigInt
0x8000000000000000n		// 2n**63n 64位整数
```

可以用`BigInt()`将常规JS数值或字符串转为`BigInt`值：

```js
BigInt(Number.MAX_SAFE_INTEGER)
let string = "123412"
BigInt(string)
```

`BigInt`的算术运算与常规JS数值的运算类似，但除法会丢弃余数并向下(向零)舍入：

```js
3000n / 997n	// 3n 商是3
3000n % 997n	// 9n 余数是9
```

相对来说，比较操作符允许混合操作数类型：

```js
1 < 2n		// true
2 > 1n		// true
0 == 0n		// true
0 === 0n	// false 类型不等
```

`Math`对象不接受`BigInt`操作数。

### 日期和时间

`Date`是对象，但也有数值表示形式，即自1970年1月1日起至今的毫秒数，也叫时间戳：

```js
let timestamp = Date.now();		// 当前时间的时间戳
let now = new Date();			// 当前时间的日期对象
let ms = now.getTime();			// 转换为毫秒时间戳
let iso = now.toISOString();	// 转换为标准格式的字符串
```

## 文本

类型`String`。字符串是16位值的不同修改的有序序列，其中每个值都表示一个Unicode字符。

JS使用Unicode字符集的UTF-16编码，所以JS字符串是无符号16位置的序列。

### 字符串字面量

```js
"string"
'string'
`string`				// 反引号 ES6新增
```

### 字符串字面量中的转义序列

| 序列     | 表示的字符                                                   |
| -------- | ------------------------------------------------------------ |
| `\0`     | NUL字符(\u0000)                                              |
| `\b`     | 退格符(\u0008)                                               |
| `\t`     | 水平制表符(\u0009)                                           |
| `\n`     | 换行符(\u000A)                                               |
| `\v`     | 垂直制表符(\u000B)                                           |
| `\f`     | 换页符(\u000C)                                               |
| `\r`     | 回车符(\u000D)                                               |
| `\"`     | 双引号(\u0022)                                               |
| `\'`     | 撇号或单引号(\u0027)                                         |
| `\\`     | 反斜杠(\u005c)                                               |
| `\xnn`   | 由2位十六进制数字`nn`指定的Unicode字符                       |
| `\unnnn` | 由4位十六进制数字`nnnn`指定的Unicode字符                     |
| `\u{n}`  | 由码点`n`指定的Unicode字符，其中`n`是介于`0`和`10FFFF`之间的1到6位十六进制数字(ES6) |

若字符`\`位于上表之外的字符前，那么该反斜杠会被忽略。

### 使用字符串

使用`+`拼接字符串。

字符串的长度：`s.length`

```js
let s = "Hello, World";

// 取字符串
s.substring(1, 4)			// 2~4个字符
s.slice(1, 4)				// 同上
s.slice(-3)					// 最后3个字符
s.split(", ")				// ["Hello", "World"]

// 搜索字符串
s.indexOf("l")				// 2 第一个字母l的位置
s.indexOf("l", 3)			// 3 位置3后面第一个"l"的位置
s.indexOf("zz")				// -1 s并不包含子串"zz"
s.lastIndexOf("l")			// 10 最后一个字母l的位置

// ES6之后版本的布尔值搜索函数
s.startsWith("Hell")		// true 字符串以Hell开头
s.endWith("!")				// false 字符串以!结尾
s.includes("or")			// true 字符串包含or

// 创建字符串的修改版本
s.replace("llo", "ya")		// "Heya, world"
s.toLowerCase()				// "hello, world"
s.toUpperCase()				// "HELLO, WORLD"
s.normalize()				// Unicode NFC 归一化 ES6新增
s.normalize("NFD")			// NFD归一化 还有NFKC和NFKD

// 访问字符串中的个别字符
s.charAt(0)					// "H" 第一个字符
s.charAt(s.lengh-1)			// 最后一个字符
s.charCodeAt(9)				// 72 指定位置的16位数值
s.codePointAt(0)			// 72 ES6 适用于码点大于16位的情形

// ES2017新增的字符串填充函数
"x".padStart(3)				// "  x" 在左侧添加空格 让字符串长度变为3
"x".padEnd(3)				// 右侧加空格
"x".padStart(3, "*")		// 左边加* 让字符串长度变为3
"x".padEnd(3, "-")			// x-- 让字符串长度变为3

// 删除空格函数 trim()是ES5有的 其它是Es2019新增
" test ".trim()				// test 删除开头和末尾的空格
" test ".trimStart()
" test ".trimEnd()

// 未分类字符串方法
s.concat("!")				// Hello, world! 可以用+操作符代替
"<>".repeat(5)				// <><><><><> 拼接5次 ES6新增
```

如`replace`和`toUpperCase`它们是返回新字符串，而不会修改字符串。

可以用下标访问字符串的指定字符：

```js
let words = "hello";
s[0]			// h
```

### 模板字面量

