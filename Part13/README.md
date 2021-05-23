# 异步JavaScript

ES6新增的期约(Promise)是一种对象，代表某个异步操作尚不可用的结果。

关键字`async`和`awiat`是ES2017引入，为简化异步编程提供了新语法，允许开发者将基于期约的异步代码写成同步的形式。

异步迭代器和`for/await`循环是ES2018引入，允许在看起来同步的简单循环中操作异步事件流。

## 使用回调的异步编程

在最基本的层面上，JS异步使用回调实现。

回调即函数，可以传给其他函数。而其他函数会在满足某个条件或发生某个异步事件时调用该函数。

回调函数被调用，相当于通知你满足了某个条件或发生了某个事件，有时这个调用还会包含函数参数，能够提供更多细节。

### 定时器

异步操作之一：在一定时间过后运行某些代码。可以用`setTimeout()`实现。

```js
// 一分钟后回调checkForUpdates()
setTimeout(checkForUpdates, 60000)
```

### 事件

客户端JS编程几乎全是事件驱动。用户在按下键盘按键、移动鼠标、单击鼠标或轻点触摸屏设备时，浏览器生成事件。

事件驱动的JS程序在特定上下文中为特定类型的事件注册回调函数，而浏览器在指定的事件发生时调用这些函数。

这些回调函数叫做事件处理程序或者事件监听器，通过`addEventListener()`注册:

```js
let okay = document.querySelector('#confirmUpdateDialog button.okay');

// 注册一个回调函数 当用户单击该按钮时触发 applyUpdate是个假想的回调函数。
okay.addEventListener('click', applyUpdate);
```

`addEventListener`第一个参数是字符串，指定注册的事件类型(此处是一次鼠标单击或轻点触摸屏)。若用户单击或轻点了那个元素，浏览器就会调用`applyUpdate()`回调函数，并给他传入一个对象，其中包含有关事件的详细信息(如事件发生的事件和鼠标指针的坐标)。

### 网络事件

另一个常见的异步操作来源是网络请求。

浏览器中运行的JS可以通过类似下述代码从Web服务器获取数据：

```js
function getCurrentVersionNumber(versionCallback) {
    // 通过脚本向后端版本API发送一个HTTP请求
    let request = new XMLHttpRequest();
    request.open("GET", "http://www.example.com/api/version");
    request.send();
    
    // 注册一个将在响应到达时调用的回调
    request.onload = function() {
        if (request.status === 200) {
            // 若HTTP状态码没问题 则取得版本号并调用回调
            let currentVersion = parseFloat(request.responseText);
            versionCallback(null, currentVersion);
        } else {
            // 否则通过回调报告错误
            versionCallback(response.statusText, null);
        }
    };
    // 注册另一个将在网络出错时调用的回调
    request.onerror = request.ontimeout = function(e) {
        versionCallback(e.type, null);
    };
}
```

客户端JS代码可以用`XMLHttpRequest`类及回调函数来发送HTTP请求并异步处理服务器返回的响应。

大多Web API可以通过在生成事件的对象上调用`addEventListener()`并将相关事件的名字传给回调函数来定义事件处理程序。也可以通过将回调函数赋值给这个对象的一个属性来注册事件监听器。如上述代码中的`onload`、`onerror`和`ontimeout`属性。

相较而言，`addEventListener()`是种更灵活的技术，因为它支持多个事件处理程序。但如果确定没有别的代码会给同一个对象的同一个事件再注册监听器，就可以简单些，将相应的属性设为你的回调。

### Node中的回调与事件

Node.js服务器端JS环境底层就是异步的，定义了很多使用回调和事件的API，例如读取文件内容的默认API就是异步的：

```js
const fs = requirs("fs");		// "fs"模块有文件系统相关的API
let options = {
    // 默认选项可以写在这里
};

// 读取配置文件 然后调用回调函数
fs.readFile("config.json", "utf-8", (err, text) => {
    if (err) {
        // 若有错误 显示一条警告信息 但仍然继续
        console.warn("Could not read config file:", err);
    } else {
        // 否则 解析文件内容并赋值给选项对象
        Object.assign(options, JSON.parse(text));
    }
    
    // 无论是什么情况 都会启动运行程序
    startProgram(options);
})
```

Node的`fs.readFile()`函数的最后一个参数是个接收两个参数的回调函数。它会异步读取指定文件，然后调用回调。若读取文件成功，则会把文件内容传给第二个参数，若失败，则将错误传给第一个参数。

下述函数展示了在Node种通过HTTP请求获取URL的内容。其包含两层处理事件监听器的异步代码。Node使用`on()`方法而非`addEventListener()`注册事件监听器：

```js
const https = require("https");

// 读取URL的文本内容 将其异步传给回调
function getText(url, callback) {
    // 对URL发送一个HTTP GET请求
    request = https.get(url);
    
    // 注册一个函数处理"response"事件
    request.on("response", response => {
        // 这个响应事件意味着收到了响应头
        let httpStatus = response.statusCocde;
        
        // 此时并没有受到HTTP响应体 因此还要再注册几个事件处理程序 以便收到响应体时被调用
        response.setEncoding("utf-8");		// 应该收到Unicode文本
        let body = "";						// 在这里累积
        
        // 每个响应体块就绪时都会调用这个事件处理程序
        response.on("data", chunk => { body += chunk; });
        
        // 相应完成时回调用这个事件处理程序
        response.on("end", () => {
            if (httpStatus === 200) {
                callback(null, body);		// 将响应体传给回调
            } else {						// 否则传错误
                callback(httpStatus, null);
            }
        });
    });
    
    // 这里也为底层网络错误注册了一个事件处理程序
    request.on("error", (err) => {
        callback(err, null);
    });
}
```

## 期约

期约是种为简化异步编程而设计的核心语言特性。是一个对象，表示异步操作的结果。该结果可能就绪也可能未就绪。

最简单的情况下，期约就是一种处理回调的不同方式。

期约标准化了异步错误处理，通过期约链提供了一种让错误正确传播的途径。

### 使用期约

假设有个`getJSON()`函数不接收回调参数，而是将HTTP响应体解析成JSON格式并返回一个期约。这里是使用这个返回期约的辅助函数的示例：

```js
getJSON(url).then(jsonData => {
    // 一个回调函数 会在解析得到JSON值后被异步调用 并接收该JSON值作为参数
})
```

`getJSON()`向指定URL发送一个异步HTTP请求，然后再请求结果待定期间返回一个期约对象。这个期约对象有个实例方法`then()`。回调函数传给了`then()`。当HTTP响应到达时，响应体会被解析为JSON格式，而解析后的值会被传给作为`then()`的参数的函数。

如果多次调用一个期约对象的`then()`方法，那么指定的每个函数都会在预期计算完成后被调用。

期约表示的是单次计算，每个通过`then()`方法注册的函数都只会被调用一次。即便调用`then()`的时候异步计算已经完成，传给`then()`的函数也会被异步调用。

最简单的语法层面，`then()`方法是期约独有的特性，而直接将`.then()`附加给返回期约的函数调用是一种惯用方法，不需要把期约对象赋给某个变量然后再另外调用`.then()`。

命名规范：以动词开头来命名：

- 返回期约的函数
- 使用期约结果的函数

#### 使用期约处理错误

异步操作通常会有多种失败原因。

对于期约，可以通过给`then()`方法传第二个函数来实现错误处理：

```js
getJSON("/api/user/profile").then(displayUserProfile, handleProfileError);
```

基于期约的异步计算在正常结束后，会把计算结果传给作为`then()`的第一个参数的函数。

基于期约的异步计算把异常传给作为`then()`的第二个参数的函数。

但这只能捕获在这个返回期约的函数中的错误，如果第一个函数引发错误，那么因为`getJSON`返回时是被异步调用的，所以也是异步执行，所以就不能明确地抛出一个异常。

实际中很少传两个参数。更好且更符合传统的错误处理方式：

```js
getJSON("/api/user/profile").then(displayUserProfile).catch(handleProfileError);
```

表明`getJSON()`正常结果还是会返回给`displayUserProfile`，但`getJSON()`和`displayUserProfile()`在执行时发生的错误都会传给`handleProfileError()`。这个`catch()`方法只是对调用`then()`时以`null`作为第一个参数，以指定的错误处理函数作为第二个参数的一种简写形式。

#### 术语

若`then()`的第一个回调函数被调用，则说期约得到"兑现"，若第二个回调被调用，则说期约被"拒绝"。若未兑现也未被拒绝，则是"待定"。一旦兑现或拒绝，则说它已经"落定"，不会再从兑现变成拒绝，反之亦然。

### 期约链

一个重要优点：以线性`then()`方法调用链的形式表达一连串异步操作，假想的期约链：

```js
fetch(documentURL)
	.then(response => response.json())
	.then(document => {return render(document)})
	.catch(error => handle(error));
```

前章中的XMLHttpRequest兑现已被更新的基于期约的Fetch API取代。这个HTTP API的最简单形式就是函数`fetch()`。传给它一个URL，返回一个期约。该期约会在HTTP响应开始到达且HTTP状态和头部可用时兑现。

这段代码中的方法调用忽略了传给方法的参数：

```js
fetch().then().then()
```

方法链↑。

每个`then`调用都会返回一个新期约对象。这个新期约对象在传给`then()`的函数执行结束才会兑现。

```js
fetch(URL)              // 任务1 返回期约1
    .then(callback1)    // 任务2 返回期约2
    .then(callback2)    // 任务3 返回期约3
```

### 解决期约

一个期约与另一个期约发生了关联。

```js
function c1(response){			// 回调1
    let p4 = response.json();
    return p4;					// 返回期约4
}

function c2(profile){			// 回调2
    displayUserProfile(profile);
}

let p1 = fetch("/api/user/profile");		// 期约1 任务1
let p2 = p1.then(c1);						// 期约2 任务2 在p1上注册c1 返回p2
let p3 = p2.then(c2);						// 期约3 任务3 在p2上注册c2 返回p3
```

当`c1`返回`p4`时，`p2`得到解决，但解决不等同于兑现，所以任务3还不会开始。只有当HTTP响应体全部可用时，`.json()`方法才可以解析它并以解析后的值兑现`p4`。`p4`兑现后，`p2`也会自动以该解析后的JSON值兑现。此时解析后的JSON对象被传给`c2`，任务3开始。

### 再谈期约和错误

细致的错误处理在异步编程中确实非常重要。

在同步代码中，如果不编写错误处理逻辑，至少会看到异常和栈追踪信息，从而能够查找出错的原因。

但在异步代码中，未处理的异常往往得不到报告，错误只会静默发生，导致它们更难调试。

#### catch和finally方法

期约的`.catch()`实际上是对以`null`为第一个参数、以错误处理回调为第二个参数的`.then()`调用的简写。

对任何期约`p`和错误回调`c`，下述两行代码是等价的：

```js
p.then(null, c);
p.catch(c);
```

使用`.catch()`的理由：

1. 更简单
2. 名字对应`try/catch`的`catch`

ES2018中，期约对象还定义了`.finally()`方法，用途类似`try/catch/finally`中的`finally`子句。

若在期约链中添加`.finally()`调用，那么传给`.finally()`的回调会在期约落定时被调用。

`.finally()`也返回一个新期约对象。但`.finally()`回调的返回值通常会被忽略，而解决或拒绝调用`finally()`的期约的值一般也会用来解决或拒绝`.finally()`返回的期约。如果`.finally()`回调抛出异常，就会用这个错误拒绝 `.finally()`返回的期约。

```js
fetch("/api/user/profile")			// p1 发送HTTP请求 
    .then(response => {				// p2 response是c1 在状态和头部就绪时调用 
        if(!response.ok) {			// 如果遇到404或类似错误
            return null;
        }
    
    	// 检查头部以确保服务器发送给我们的是JSON
    	// 如果不是 说明服务器坏了 这是个严重错误
    	let type = response.headers.get("content-type");
    	if (type !== "application/json") {
            throw new TypeError(`Expected JSON, got ${type}`);
        }
    
    	// 如果到这里了 说明状态码是2xx 内容类型也是JSON 那我们就能返回一个期约 表示解析响应体之后得到的JSON对象
    	return response.json();
	})
	.then(profile => {			// p3 profile是c2 调用时传入解析后的响应体或null
    	if(profile) {
            displayUserProfile(profile);
        }
    	else {					// 如果遇到了404错误并返回null 则会走到这里
            displayLoggedOutProfilePage();
        }
	})
	.catch(e => {				// e是c3
    	if(e instanceof NetworkError) {
            // fetch()在互联网连接故障时会走到这里
            displayErrorMessage("Check your internet connection.");
        }
    	else if (e instanceof TypeError) {
            // 在上面抛出TypeError时会走到这里
            displayErrorMessage("Something is wrong with our server!");
        }
		else {
            // 走到这里说明发生了意料之外的错误
            console.error(e);
        }
	});
```

第一种可能失败的情况：

1. `fetch()`出现网络连接故障，
2. 期约`p1`会以一个`NetworkError`对象被拒绝。
3. 因为没给`.then()`调用传错误处理回调函数作为第二个参数，所以`p2`也会以同一个`NetworkError`对象被拒绝。
4. 出于与`p2`相同理由，`p3`也被拒绝。
5. `c3`错误处理回调被调用，其中特定于`NetworkError`的代码会运行。

另一种失败方式：

1. HTTP请求返回404或其它HTTP错误。因为它们是有效的HTTP响应，所以`fetch()`不认为它们错误。
2. `fecth()`将它封装在`Response`对象中并以该对象兑现`p1`。
3. `p1`对象导致`c1`被调用。
4. `c1`中代码检查`Response`的`ok`属性，检测到为正常HTTP响应就会返回`null`。由于该返回值(`null`)非期约，所以立即兑现`p2`，`c2`调用。`c2`代码显示检查输入值是否为假，以此向用户显示不同的结果。

如果Content-Type头设置的不对，那就会在`c1`中抛出`TypeError`，从而导致`p2`以该`TypeError`对象被拒绝，因为没给`p2`传错误处理程序，所以`p3`也被拒绝。此时不会调用`c2`，`TypeError`直接传给`c3`，其中包含显式检查和处理这种错误的代码。

上诉注意：该错误对象是以常规、同步`throw`语句抛出，而该错误最终在期约链中被一个`.catch()`方法调用处理。

如果期约链的某个环节回因错误而失败，而该错误属于某种可恢复的错误，那就不应该停止后续环节代码的运行，可以在链中插入一个`.catch()`：

```js
startAsyncOperation()
    .then(doStageTwo)
    .catch(recoverFromStageTwoError)
    .then(doStageThree)
    .then(doStageFour)
    .catch(logStageThreeAndFourErrors);
```

传给`.catch()`的回调只有在上一环的回调抛出错误时才会被调用。如果回调正常返回，那么该`.catch()`回调就会被跳过，之前回调返回的值会成为下个`.then()`回调的输入。

`.catch()`也可以处理错误并从错误中恢复。`.catch()`回调可以抛出新错误，但如果正常返回，那这个返回值就会用于解决或兑现与之关联的期约，从而停止错误传播。

**实际开发中，忘记从回调函数中返回值是导致期约相关问题的常见原因。**

### 并行期约

