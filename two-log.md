## index.js

> two-log 源


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


``` js
'use strict';

Object.defineProperty(exports, '__esModule', {
	value: true,
}); // es6 export default
const winston = require('winston');
const Ora = require('ora');
const pkgName = require('get-module-name').sync();

// two-log
let D = false; // default no debug
let LOGGER = null; // main
let LOCK = false; // only one set debug
let LoggerNAME = 'ora'; // for test

// log
let defaultWinston = {
	level: 'debug',
	transports: [
		new winston.transports.Console({
			datePattern: '.yyyy-MM-ddTHH-mm',
			colorize: true,
		}),
		new winston.transports.File({
			filename: `${pkgName}.log`,
			handleExceptions: true,
			maxsize: 52000,
			maxFiles: 1,
			level: 'info',
			colorize: true,
		}),
	],
};
let logOpts = null; // winston options merge user options

// two-log

const twoLog = (debug = false, userUse) => {
	if (typeof debug !== 'boolean') {
		throw new TypeError(`Expected a string, got ${typeof debug}`);
	} else {
		if (LOCK) {
			throw new TypeError(`Set two-log debug just only one,❌`);
		}

		D = debug;
		LoggerNAME = D ? 'log' : 'ora';
		LOCK = true;
	}

	userUse && userUse(API);

	return {
		start: loggerStart,
		text: loggerText,
		stop: loggerStop,
		one: oneOra,
	};
};

// user use API
let API = {
	ora: Ora,
	log: winston,
	setLog: options => {
		logOpts = Object.assign({}, defaultWinston, options);
	},
};

// for test
const _UNLOCK = function() {
	LOCK = false;
};

/**
 * @description one time ora spinner
 * @param {string} str
 * @param {any} options
 * @param {string} options.color yellow
 * @param {string} options.end succeed
 */
function oneOra(str, options) {
	let { color, end } = Object.assign(
		{ color: 'yellow', end: 'succeed' },
		options
	);

	if (LOGGER && !D) {
		let l = LOGGER;
		let oldColor = l.color;
		let oldText = l.text;
		l.color = color;
		l.text = str;

		if (str && end) {
			l[end](str);
		} else {
			l.stop();
		}

		LOGGER = Ora(oldText).start();
		LOGGER.color = oldColor;
	} else if (!D && !LOGGER) {
		let l = Ora(str).start();
		l.color = color;

		if (str && end) {
			l[end](str);
		} else {
			l.stop();
		}

		l = null;
	} else {
		return false;
	}
	return true;
}

/**
 * @description start logger
 * @param {string} str
 * @param {string} options.ora ora color
 * @param {string} options.log winston log level
 * @param {string} options.only only one {ora|log}
 */
function loggerStart(str, options) {
	logOpts = logOpts ? logOpts : defaultWinston;

	let { ora, log, only } = Object.assign(
		{ ora: 'yellow', log: 'debug' },
		options
	);
	let res = ' '; // for test

	if (!D && (!only || only == 'ora')) {
		LOGGER = Ora(str).start();
		LOGGER.color = ora;

		res += 'start'; // for test
	} else if (D && (!only || only == 'log')) {
		LOGGER = new winston.Logger(logOpts);
		LOGGER[log](str);

		res += log; // for test
	}

	return LoggerNAME + res; // for test
}

/**
 * @description set logger text
 * @param {String} str
 * @param {string} options.ora ora color
 * @param {string} options.log winston log level
 * @param {string} options.only only one {ora|log}
 */
function loggerText(str, options) {
	if (!LOGGER) {
		return false;
	}
	let { ora, log, only } = Object.assign(
		{ ora: 'yellow', log: 'debug' },
		options
	);

	let res = ' '; // for test

	if (!D && (!only || only == 'ora')) {
		LOGGER.text = str;
		LOGGER.color = ora;

		res += 'text'; // for test
	} else if (D && (!only || only == 'log')) {
		LOGGER[log](str);
		res += log; // for test
	}
	return LoggerNAME + res; // for test
}

/**
 * @description logger stop
 * @param {string} str
 * @param {string} options.ora ora {fail|succeed|warn} https://github.com/sindresorhus/ora#instance
 * @param {string} options.log winston log level
 * @param {string} options.only only one {ora|log}
 */
function loggerStop(str, options) {
	if (!LOGGER) {
		return false;
	}
	let { ora, log, only } = Object.assign({ ora: '', log: 'debug' }, options);

	let res = ' '; // for test

	if (!D && (!only || only == 'ora')) {
		if (ora && str) {
			LOGGER[ora](str);
			res += ora; // for test
		} else {
			LOGGER.stop();
			res += 'stop'; // for test
		}
	} else if (D && (!only || only == 'log')) {
		if (str) {
			LOGGER[log](str);
		}
		res += log; // for test
	}

	LOGGER = null;
	logOpts = null;

	return LoggerNAME + res; // for test
}

exports = module.exports = twoLog;
exports.twoLog = twoLog;
exports.loggerStart = loggerStart;
exports.loggerText = loggerText;
exports.loggerStop = loggerStop;
exports.oneOra = oneOra;
exports._UNLOCK = _UNLOCK;
exports.default = twoLog;
```
