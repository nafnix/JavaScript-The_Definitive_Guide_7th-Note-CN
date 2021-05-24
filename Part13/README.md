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

有时希望并行执行多个异步操作。函数`Promise.all()`可以做到这一点。`Promise.all()`接收一个期约对象的数组作为输入，返回一个期约。若输入期约中的任意一个拒绝，返回的期约也将拒绝，否则返回的期约会以每个输入期约兑现值的数组兑现。

```js
// 先定义一个URL数组
const urls = [ /* 零个或多个URL */ ]

// 然后将它转为一个期约对象的数组
promise = urls.map(url => fetch(url).then(r => r.text()));

// 用一个期约来并行运行数组中的所有期约
Promise.all(promises)
    .then(bodies => { /* 处理得到的字符串数组 */ })
    .catch(e => console.error(e));
```

`Promise.all()`实际输入数组可以包含期约对象和非期约值，非期约值会被当成一个已兑现期约的值。

ES2020中，`Promise.allSettled()`也接收一个输入期约的数组，但它永远不拒绝返回的期约，而是等所有输入期约全部落定后兑现。返回的期约解决为一个对象数组，其中每个兑现都对应一个输入期约，且都有一个`status`属性，值为`fulfilled`或`rejected`。若`status`是`fulfilled`，那么该对象还有个`value`属性，包含兑现的值。如果`status`属性值为`rejected`，那么该对象还有个`reason`属性，包含对应期约的错误或拒绝理由：

```js
Promise.allSettled([Promise.resolve(1), Promise.reject(2), 3]).then(results => {
    results[0]		// => { status: "fulfilled", value: 1 }
    results[1]		// => { status: "rejected", reason: 2 }
    results[2]		// => { status: "fulfilled", value: 3}
})
```

如果相同时运行多个期约，但只关心一个兑现的值，可以用`Promise.race()`，它返回一个期约，该期约会在输入数组中的期约有个兑现或拒绝时马上兑现或拒绝，若数组有个非期约值也会直接将其返回。

### 创建期约

#### 基于其它期约的期约

```js
// 初始期约
function getJSON(url) {
    return fetch(url).then(response => response.json());
}

// 基于期约的期约
function getHighScore() {
    return getJSON("/api/user/profile").then(profile => profile.highScore);
}
```

#### 基于同步值的期约

实现一个已有的基于期约的API，并从一个函数返回期约，尽管要执行的计算实际上并不涉及异步操作。可以用下面两个静态方法：

1. `Promise.resolve()`：接收一个值作为参数，返回一个立即(但异步)以该值兑现的期约
2. `Promise.reject()`：接收一个参数，返回一个以该参数作为理由拒绝的期约

这两个静态方法返回的期约在返回时未兑现或拒绝，但会在当前同步代码块运行结束后立即兑现或拒绝。通常会在几毫秒后发生，除非有很多特定的异步任务等待运行。

 基于期约的函数其中值是同步计算得到的，用`Promise.resolve()`异步返回是可能的，但不常见。不过在一个异步函数里包含同步执行的代码，通过`Promise.resolve()`和`Promise.reject()`来处理这些同步操作的值倒是相当常见。

#### 从头开始创建期约

可以用`Promise()`构造函数来创建一个新期约对象。

`Promise()`构造函数：

- 参数：
  - 函数：
    - 函数参数：
      - `resolve`
      - `reject`

构造函数同步调用我们的函数并为其函数参数传入对应的值。调用函数后，`Promise()`返回新创建的期约。这个期约由传给`Promise()`的函数控制。

我们传递的函数应该执行某些异步操作，然后调用`resolve`函数解决或兑现返回的期约，或者调用`reject`函数拒绝返回的期约。也不一定非要执行异步操作，可以同步调用`resolve`或`reject`，但此时创建的期约仍会异步解决、兑现或拒绝。

示例`wait()`：

```js
function wait(duration) {
    // 创建并返回新期约
    return new Promise((resolve, reject) => {	// 两个函数控制期约
        // 若参数无效 拒绝期约
        if (duration < 0) {
            reject(new Error("Time travel not yet implemented"));
        }
        // 否则 异步等待 然后解决期约
        // setTimeout调用resolve()时未传参, 这意味新期约会以undefined值兑现
        setTimeout(resolve, duration);
    });
}
```

如果把一个期约传给`resolve()`，返回的期约会解决未该新期约。但一般都是传非期约值，该值会兑现返回的期约。

### 串行期约

使用`Promise.all()`可以并行执行任意数量的期约，而期约链可以表达一连串固定数量的期约。但是按顺序运行任意数量的期约有些棘手。但如果我们有个要抓取的URL数组，为避免网络过载，想一次只抓一个URL。如果该数组是任意长度，内容未知，那就不能提前写出期约链，而要像如下代码构建：

```js
function fetchSequentially(urls) {
    // 抓取URL时 要将响应体存在里面
    const bodies = [];
    
    // 这个函数返回一个期约 只抓取一个URL响应体
    function fetchOne(url) {
        return fetch(url)
            .then(response => response.text())
            .then(body => {
                // 将响应体存于数组中 此处故意省略返回值
                bodies.push(body);
        });
    }
    
    // 从一个立即(以undefined值)兑现的期约开始
    let p = Promise.resolve(undefined);
    
    // 现在循环目标URL 构建任意长度的期约链 链的每个环节都会拿取一个URL的响应体
    for(url of urls) {
        p = p.then(() => fetchOne(url));
    }
    
    // 期约链的最后一个期约兑现后 响应体数组(bodies)也已就绪 所以可以将该bodies数组通过期约返回
    return p.then(() => bodies);
}
```

`fetchSequentially`先创建一个返回后立即兑现的期约。然后基于这个初始期约构建一个线性的长期约链并返回链中的最后一个期约。类似多米诺骨牌，推倒第一张。

还有种类似套娃那样一系列相互嵌套的期约，让每个期约的回调创建并返回下个期约：

```js
// 该函数接收一个输入值数组和一个promiseMaker函数
// 对输入数组中的任何值x promiseMaker(x)都应该返回
// 一个兑现未输出值的期约 该函数返回一个期约 该期约最终会兑现为一个包含计算得到的输出值的数组

// promiseSequence()每次只运行一个期约 直到上个期约兑现之后才调用promiseMaker()计算下个值
function promiseSequence(inputs, promiseMaker) {
 
    inputs = [...inputs];						// 为数组创建一个可以修改的私有副本
    
    // 这是要用作期约回调的函数 它的伪递归魔术是核心逻辑
    function handleNextInput(outputs) {
        if (inputs.length === 0) {
            // 若没有输入值 则返回输出值的数组
            // 该数组最终兑现该期约 以及前面所有已经解决但未兑现的期约
            return outputs;
        } else {
            // 若还有要处理的输入值 则返回一个期约对象 将当前期约解决为一个来自新期约的未来值
            let nextInput = inputs.shift();		// 取得下个输入值
            return promiseMaker(nextInput)		// 计算下个输出值 然后用这个新输出值创建一个新输出值的数组
                .then(output => outputs.concat(output))
                // 然后"递归" 传入新的、更长的输出值的数组
                .then(handleNExtInput);
        }
    }
    // 从一个以空数组兑现的期约开始 使用上面的函数作为它的回调
    return Promise.resolve([]).then(handleNextInput);
}
```

使用它抓取多个URL的响应示例：

```js
// 传入一个URL 返回一个以该URL的响应体文本兑现的期约
function fetchBody(url) { return fetch(url).then(r => r.text()); }
// 使用它依次抓取一批URL的响应体
promiseSequence(urls, fetchBody).then(bodies => { /* 处理字符串数组 */}).catch(console.error);
```

## async和await

ES2017新增`async`和`await`，代表异步JS编程范式的迁移。

`async`和`await`接收基于期约的高效代码并且隐藏期约，让我们的异步代码像低效阻塞的同步代码一样容易理解和推理。

### await表达式

`await`关键字接收一个期约并将其转为一个返回值或一个抛出的异常。给定一个期约`p`，表达式`await p`会一直等到`p`落定。

通常将`await`放在一个会返回期约的函数前调用：

```js
let response = await fetch("/api/user/profile");
let profile = await response.json();
```

`await`关键字不会导致我们的程序阻塞或在指定的期约落定前什么都不做。代码仍是异步，`await`只是掩盖该事实。

### async函数

因为任何使用`await`的代码都是异步的，所以导致：只能在以`async`关键字声明的函数内使用`await`关键字。

```js
async function getHighScore() {
    let response = await fetch("/api/user/profile");
    let profile = await response.json();
    return profile.highScore;
}
```

将函数声明为`async`意味该函数的返回值将是个期约，即使函数体内不出现期约相关的代码。

因为`getHighScore`返回期约，所以可以对它使用`await`：

```js
displayHighScore(await gethighScore());
```

不过上面这行代码也只有它在另一个`async`函数里才可以运行。在顶部必须以常规方式来处理返回的期约：

```js
getHighScore().then(displayHighScore).catch(console.error);
```

可以对任何函数用`async`关键字。

### 等候多个期约

假设使用`async`重写`getJSON()`：

```js
async function getJSON(url) {
    let response = await fetch(url);
    let body = await response.json();
    return body;
}
```

使用：

```js
let value1 = await getJSON(url1);
let value2 = await getJSON(url2);
```

上述代码的问题在于不必顺序执行。这样写意味着必须等到`value1`得到结果才开始抓取`value2`。如果第二个值不依赖第一个值，那应该可以同时抓取两个值。

要等候一组并发执行的`async`函数，可以像使用期约一样直接用`Promise.all()`：

```js
let [value1, value2] = await Promise.all([getJSON(url1), getJSON(url2)]);
```

### 实现细节

`async`：

```js
async function f(x) { /* 函数体 */ }
```

可以将其想象成一个返回期约的包装函数，包装了原始函数的函数体：

```js
function f(x) {
    return new Promise(function(resolve, reject) {
        try {
            resolve((function(x) { /* 函数体 */ })(x));
        }
        catch(e) {
            reject(e);
        }
    });
}
```

## 异步迭代

## for/await提供

Node 12的可读流实现了异步可迭代。可以像下面这样用`for/await`循环从一个流中读取连续的数据块：

```js
const fs = require("fs");

async function parseFile(filename) {
    let stream = fs.createReadStream(filename, { encoding: "utf-8" });
	for await (let chunk of stream) {
        parseChunk(chunk);			// 假设parseChunk()是在其他地方定义的
    }
}
```

与常规的`await`表达式类似，`for/await`循环也是基于期约的。

大体上说，此处的异步迭代器会产生一个期约，而`for/await`循环等待该期约兑现，将兑现值赋给循环变量，然后再运行循环体。之后再从头开始，从迭代器取得另一个期约并等待这个新期约兑现。

假如希望第一次抓取的结果尽快可用，不想因此而等待抓取其它URL(当然也有可能第一次抓取的时间是最长的，因此这样不一定比用`Promise.all()`更快)。数组是可迭代的，所以我们可以用常规的`for/of`循环来迭代该期约数组：

```js
for(const promise of promises) {
    response = await promise;
    handle(response);
}

// 同上
for await (const response of promises) {
    handle(response);
}
```

这两个示例都只能在以`async`声明的函数内部才能使用。

### 异步迭代器

`for/await`与常规迭代其兼容，但更适合异步可迭代对象，会在尝试`Symbol.iterator`前先尝试`Symbol.asyncIterator`方法。

异步迭代器与常规迭代器的重要区别：

1. 异步可迭代对象以符号名字`Symbol.asyncIterator`而非`Symbol.iterator`实现了一个方法
2. 异步迭代器的`next()`方法返回一个期约，解决为一个迭代器结果对象，而非直接返回一个迭代器结果对象

对一个常规同步可迭代的期约数组使用`for/await`时，操作的是同步迭代器结果对象。其中，`value`属性是个期约对象，但`done`属性是同步值。真正的异步迭代器返回的是迭代结果对象的期约，其中`value`和`done`都是异步值。

对于异步迭代器，关于迭代何时结束的选择可以异步实现。

### 异步生成器

对于异步迭代器，可以用声明为`async`的生成器函数实现。

声明为`async`的异步生成器同时具有异步函数和生成器的特性，也就是可以像在常规异步函数中一样使用`await`，也可以像在常规生成器中一样用`yield`。但通过`yield`生成的值会自动包装到期约中。

示例使用异步生成器和`for/await`循环，通过循环代码而非`setInterval()`回调函数实现以固定时间间隔重复运行代码：

```js
// 基于期约的包装函数 包装setTimeout()以实现等待
// 返回一个期约 这个期约会在指定的毫秒数之后兑现
function elapsedTime(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
}

// 一个异步迭代器函数 按照固定的时间间隔
// 递增并生成指定个数的计算器
async function* clock(interval, max=Infinity) {
    for (let count = 1; count <= max; count++) {	// 常规for循环
		await elapsedTime(interval);				// 等待时间流逝
		yield count;								// 生成计数器
	}
}

// 一个测试函数 使用异步迭代器和for/await
async function test() {								// 使用async声明 以便使用for/await
    for await (let tick of clock(300, 100))				// 循环100次 每次间隔300ms
        console.log(tick);
}
```

### 实现异步迭代器

除了用异步生成器实现异步迭代器，还可以用`Symbol.asynciterator`实现。

需要定义一个包含`Symbol.asynciterator()`方法的对象，该方法要返回一个包含`next()`方法的对象，而`next()`方法要返回解决为一个迭代器结果对象的期约。

重写`clock`，返回一个异步可迭代对象：

```js
function clock(interval, max=Infinity) {
    // 一个setTimeout的期约版 可以实现等待
    // 注意参数是个绝对时间而非时间间隔
    function until(time) {
        return new Promise(resolve => setTimeout(resolve, time - Date.now()));
    }
    
    // 返回一个异步可迭代对象
    return {
        startTime: Date.now(),			// 记住开始时间
        count: 1,						// 记住第几次迭代
        async next() {					// 使其成为迭代器
            if (this.count > max) {		// 该结束了吗
                return { done: true };	// 表示结束的迭代结果
            }
            // 计算下次迭代什么时间开始
            let targetTime = this.startTime + this.count * interval;
            // 等待该时间到来
            await until(targetTime);
            // 在迭代结果对象中返回计数器的值
            return { value: this.count++ };
        },
        // 该方法意味着这个迭代器对象同时也是个可迭代器对象
        [Symbol.asyncIterator]() { return this; }
    };
}
```

如果不在`for/await`循环里用异步迭代器，那可以在任何时候调用`next()`，对于基于生成器的`clock()`版本，如果连续调用3次`next()`方法，就可以得到3个期约，而这3个期约将几乎同时兑现。在这里实现的这个基于迭代器的版本则没有这个问题。

异步迭代器的优点：允许我们表示异步事件流或数据流。

在面对某些异步源时，比如事件处理程序的触发，要实现异步迭代器会比较困难。因为通常只有一个事件处理程序响应事件，但每次调用迭代器的 `next()`都必须返回独一无二的期约对象，而在第一个期约解决前很有可能出现多次调用`next()`的情况。这意味着任何异步迭代器方法都必须能在内部维护一个期约队列，让这些期约按照异步事件发生的顺序依次解决。

示例`AsyncQueue`类，包含队列类的`enqueue()`和`dequeue()`方法。其中`dequeue()`方法返回一个期约而非实际值。意味在尚未调用`enqueue()`前调用`dequeue()`前是没有问题的。

`AsyncQueue`类的实现直接使用期约。

```js
/*
 * 一个异步可迭代队列类 使用enqueue()添加值
 * 使用dequeue()移除值 dequeue()返回一个期约
 * 意味值可以在入队前出队 因而可以与for/await循环一起配合使用(这个循环在调用close()方法前不会终止)
 */

class AsyncQueue {
    constructor() {
        // 已经入队尚未出队的值保存在这里
        this.values = [];
        // 如果期约出队时它们对应的值尚未入队 就把那些期约的解决方法保存在这里
        this.resolvers = [];
        // 一旦关闭 任何值都不能再入队 也不会再返回任何未兑现的期约
        this.closed = false;
    }
    
    enqueue(value) {
        if (this.closed) {
            throw new Error("AsyncQueue closed");
        }
        if (this.resolvers.length > 0) {
            // 如果这个值已经有对应的期约 则解决该期约
            const resolve = this.resolvers.shift();
            resolve(value);
        }
        else {
            // 否则 让其排队
            this.values.push(value);
        }
    }
    
    dequeue() {
        if (this.values.length > 0) {
            // 如果有个排队的值 为它返回一个解决期约
            const value = this.values.shift();
            return Promise.resolve(value);
        }
        else if (this.closed) {
            // 如果没有排队的值 而且队列已关闭
            // 返回一个解决为EOS(流终止)标记的期约
            return Promise.resolve(AsyncQueue.EOS);
        }
        else {
            // 否则 返回一个未解决的期约 将解决方法排队 以便后面使用
            return new Promise((resolve) => { this.resolvers.push(resolve); });
        }
    }
    
    close() {
        // 一旦关闭 任何值都不能再入队
        // 因此以EOS标记解决所有待决期约
        while(this.resolvers.length > 0) {
            this.resolvers.shift()(AsyncQueue.EOS);
        }
        this.closed = true;
    }
    
    // 定义这个方法 让这个类成为异步可迭代对象
    [Symbol.asyncIterator]() { return this; }
    
    // 定义该方法 让这个类成为异步迭代器
    // dequeue()返回的期约会解决为一个值
    // 或者在关闭时解决为EOS标记。这里需要返回一个解决为迭代器结果对象的期约
    next() {
        return this.dequeue().then(value => (value === AsyncQueue.EOS 
                                             ? { value: undefined, done: true}
                                             : { value: value, done: false });
    }
}

// dequeue() 方法返回的标记值 在关闭时表示 "流终止"
AsyncQueue.EOS = Symbol("end-of-stream");
```

示例使用`AsyncQueue`产生一个浏览器事件流，可以通过`for/await`循环来处理：

```js
// 把指定文档元素上指定类型的事件推入一个AsyncQueue对象
// 然后返回该队列 以便将其作为事件流来使用
function eventStream(elt, type) {
    const q = new AsyncQueue();						// 创建一个队列
    elt.addEventListener(type, e=>q.enqueue(e));	// 入队事件
    return q;
}

async function handleKeys() {
	// 取得一个keypress事件流 对每个时间都执行依次循环
    for await (const event of eventSteam(document, "keypress")) {
        console.log(event.key);
    }
}
```

