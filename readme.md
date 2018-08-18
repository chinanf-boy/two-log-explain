# two-log [![explain]][source] [![translate-svg]][translate-list]

[explain]: http://llever.com/explain.svg
[source]: https://github.com/chinanf-boy/Source-Explain
[translate-svg]: http://llever.com/translate.svg
[translate-list]: https://github.com/chinanf-boy/chinese-translate-list
    
「 我只需要两种信息输出方式 1. `ora` 2. `winston` 」

[可以先用一下](https://github.com/chinanf-boy/two-log#usage)

---

## explain 🀄️

<!-- doc-templite START generated -->
<!-- time = '2018 8.15' -->
<!-- name = 'chinanf-boy' -->
<!-- repo = 'two-log' -->
<!-- commit = 'v0.0.11' -->
版本 | 与日期 | 最新更新 | 更多
---|---|---|---
[v0.0.11][commit] | ⏰ 2018 8.15 | ![version] | [源码解释][source]

[commit]: https://github.com/chinanf-boy/two-log/tree/v0.0.11
[version]: https://img.shields.io/npm/v/two-log.svg

<!-- doc-templite END generated -->

### 贡献

欢迎 👏 勘误/校对/更新贡献 😊 [具体贡献请看](https://github.com/chinanf-boy/chinese-translate-list#贡献)

## 生活

[help me live , live need money 💰](https://github.com/chinanf-boy/live-need-money)

---

<!-- START doctoc -->
<!-- END doctoc -->

## 日志

### 可以让人明白内部运行的情况

那么 第一种日志, 就是使用[winston](https://github.com/winstonjs/winston) 「关于一切的日志」

### 但是, 但多数使用的人并不想看到那么多的输出

于是, 第二种日志, 就是使用等待 转圈圈的[ora](sindresorhus/ora) 优雅的终端等待

### 其实, 还可以使用主要给作者自身看的

第三种调试 [debug](https://github.com/visionmedia/debug) 需要设置node运行时的环境变量, 以达到相关调试

信息的输出, 这种日志其实可以与 第一种日志 换一下也可以

> 但这更像是流程查看, 也许下次吧

## 命令行触发

### 是否调试模式

命令行获取用户的 `-D`或其他值的定义,来选择 日志类型

``` js
const twoLog = require('two-log');
let D = cli.flags['D'] ? true : false;
let l = twoLog(D);
```

## 主要有三个生命周期方式

> 主要是为了对应 `ora`, 而`winston`也就是不同样子的`console.log`

- `start` 开始
- `text`  换字
- `stop`  结束 

> 我们来看看对应日志的三个阶段

### ora的角度

> [官方例子](https://github.com/sindresorhus/ora#usage)

``` js
const ora = require('ora');

const spinner = ora('Loading unicorns').start(); // start 🎩 字, 还有颜色

setTimeout(() => {
	spinner.color = 'yellow';
	spinner.text = 'Loading rainbows';  // text 阶段 可以换字, 还有颜色
}, 1000);
```

- stop 阶段, 有几种可能性

``` js
spinner.stop() // 就没了, 换个行

spinner.succeed([text]) // 打个勾 ✅, 结束

spinner.fail([text]) // 打个差 ❌, 结束

spinner.warn([text]) // ⚠️, 结束
```

### winston的角度

这个库的日志, 在三个阶段, 其实差不多一样的

> 只是选择哪种等级的输出而已

``` js
const levels = { 
  error: 0, 
  warn: 1, 
  info: 2, 
  verbose: 3, 
  debug: 4, 
  silly: 5 
};
```

好了, 看了两种日志的表现, 我们进入库吧

## package.json

``` js
"main": "dist/two-log.js",
"bin": "cli.js"，
"scripts": {
    "test": "nyc ava -v -s",
    "prepublish": "npm run build",
    "build": "microbundle",
```

命令行`cli.js`文件只是一个demo, 让你[自观感受](https://github.com/chinanf-boy/two-log#demo)

> 我用了`microbundle`压缩构建, `index.js`是源代码

## index.js

- [ ] [explain two-log](./two-log.md)

简单用文字讲讲:

1. 导入两个库

2. 因为是二选一, 所有准备 文件的全局变量`LOGGER`, 和是否调试的全局变量`D`

3. 初始化`two-log`, 因为`winston`的初始化有需要自定义, 

3.1 所以提供api给用户

``` js
let API = {
	ora: Ora, // Ora 源
	log: winston, // winston 源
	setLog: options => {
		logOpts = Object.assign({}, defaultWinston, options); // 以及可以 覆盖默认的选项 的函数
	},
};
```

3.2 一个命令行只需要一个日志,所以我上锁了

``` js
		if (LOCK) {
			throw new TypeError(`Set two-log debug just only one,❌`);
		}
```

> 当然这也是为了, 日志的生命与程序的生命周期联系在一起, 一起开始, 一起结束

3.3 并返回 log对象


``` js
// log对象, 当然你可以用其他变量
	return {
		start: loggerStart,
		text: loggerText,
		stop: loggerStop,
		one: oneOra,
    };
```

> `oneOra`是为了只显示一次 正确/错误/警告 的信息

4. loggerStart-根据用户与初始化, 拿到唯一的日志,并存入`LOGGER`

> loggerStart === log.start

``` js
/**
 * @description start logger
 * @param {string} str
 * @param {string} options.ora ora color
 * @param {string} options.log winston log level
 * @param {string} options.only only one {ora|log} 其中这一个选项, 可以自己设置只能是哪种日志能被创建
 */
function loggerStart(str, options) {
```

> 创建 是 根据`D`与`only`当然还有`LOGGER`是否存在的值 决定的

5. loggerText 换文本与或颜色

``` js
/**
 * @description set logger text
 * @param {String} str
 * @param {string} options.ora ora color
 * @param {string} options.log winston log level
 * @param {string} options.only only one {ora|log} 其中这一个选项, 可以自己设置只能是哪种日志能被输出
 */
function loggerText(str, options) {
```

> 文本 是 根据`D`与`only`当然还有`LOGGER`是否存在的值 决定的

6. loggerStop 结束日志与清空全局变量

``` js
/**
 * @description logger stop
 * @param {string} str
 * @param {string} options.ora ora {fail|succeed|warn} https://github.com/sindresorhus/ora#instance 表现一下程序的结果
 * @param {string} options.log winston log level
 * @param {string} options.only only one {ora|log} 其中这一个选项, 可以自己设置只能是哪种日志能被输出
 */
function loggerStop(str, options) {
```

因为是结束, 总要打个东西, 说明程序是否正确

> 结束 是 根据`D`与`only`当然还有`LOGGER`是否存在的值 决定的

散花 🌹