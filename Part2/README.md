# 词法结构

## JavaScript程序的文本

- 区分大小写
- 忽略记号间的空格
- 将制表符、各种ASCII控制符和Unicode间格识别为空格

## 注释

`//`和`/**/`

多行美观：

```js
/*
 * 1
 * 2
 */
```

## 字面量

字面量(literal)

## 标识符和保留字

标识符：用于在JS代码里命名常量、变量、属性、函数、类，以及为某些循环提供标记(label)。

开头：

- 字母
- 下划线
- $

## Unicode

JS程序用Unicode字符编写。

### Unicode转义序列

可以使用ASCII字符表示Unicode字符，例如`\u00E9`会被转成unicode对应的字符。

ES6新增，支持大于16位的Unicode码点，如表情符号：

```javascript
console.log("\u{1F600}");		// 打印一个笑脸
```

### Unicode归一化

若在程序中使用了非ASCII字符，如`é`可以被编码成`\u00E9`，也可以被编码成一个常规ASCII字符`e`后跟一个重音组合标记`\u0301`。这两种编码在文本编辑器中看起来完全相同，但他们的二进制编码不同，所以JS认为它们不同，这可能导致麻烦的问题。

若想在JS程序中使用Unicode字符，应保证使用自己的编辑器或其他工具对自己的源代码执行Unicode归一化，以防其中包含看起来一样但实际不同的标识符。

## 可选的分号

两行可以省略分号，一行写完就要加分号。
