---
layout: post
category : node.js
title: "用 Node.js 开发命令行工具"
tagline: "Supporting tagline"
tags : []
---

假设我们要开发一个 `hacker` 命令，在 shell 中运行 `hacker` 命令的效果：

```
hacker
> Hello, hacker!
```

首先初始化一个 npm 项目：

```
npm init
```

新建一个 index.js，内容如下：

```javascript
#!/usr/bin/env node
console.log('Hello, hacker!');
```

第一行称作：[shebang](https://en.wikipedia.org/wiki/Shebang_(Unix))，用来告诉 shell 该如何执行该脚本。

然后告诉 npm 你的命令是啥以及去哪里找可执行脚本。修改 package.json，添加一个 `bin` 属性，如下：

```
...
"bin":{
	"commandName":"./index.js"
}
...
```
键 `commandName` 就是在 shell 中使用的命令。值是相对于 package.json 文件的路径。

全局安装之后就可以使用了：

```
npm install -g
```

`npm install -g` 会将脚本链接到 PATH 的某个目录

```
$ which hakcer
> /usr/local/bin/hacker
$ readlink /usr/local/bin/hacker
> ../lib/node_modules/hacker/index.js
```

在开发阶段，使用 [npm link](https://docs.npmjs.com/cli/link) 更方便些。

## 处理命令行参数

最简单的可以通过 `process.argv`。也可以使用一些封装好的包，例如：

- [commander](https://github.com/tj/commander.js)
- [yargs](https://github.com/yargs/yargs)
- [minimist](https://github.com/substack/minimist)

其中 `yargs` 不仅可以用来处理命令行参数，它还可以：

- 配置命令行参数（是否必选、默认值、提示等）
- 设置 Git 风格的子命令
- 显示各种类型帮助信息（用法格式、实例等）

## 提示用户输入
另一种从用户获取内容的方式是从标准输入读取。简单的可以使用 `process.stdin`。同样可以使用一些封装好的包，例如：

- [inquirer](https://github.com/SBoudrias/Inquirer.js)
- [co-prompt](https://www.npmjs.com/package/co-prompt)
- [prompt](https://github.com/flatiron/prompt)

## 错误处理
根据 Unix 传统，程序执行成功返回 0，否则返回 1 。

```javascript
if (err) {
  process.exit(1);
} else {
  process.exit(0);
}
```

## 新建进程
脚本可以通过 `child_process` 模块新建子进程，用于执行 Unix 系统命令。也可以使用封装好的包，例如：

- [shelljs](https://www.npmjs.com/package/shelljs)

## 优化

### 彩色输出

- [chalk](https://www.npmjs.com/package/chalk)

### 进度条

- [progress](https://www.npmjs.com/package/progress)

## 其它

### 重定向
Unix 允许程序之间使用管道重定向数据。

```
$ ps aux | grep 'node'
```
脚本可以通过监听标准输入的 `data` 事件，获取重定向的数据。

```javascript
process.stdin.resume();
process.stdin.setEncoding('utf8');
process.stdin.on('data', function(data) {
  process.stdout.write(data);
});
```
下面是用法：

```
$ echo 'foo' | ./hello
hello foo
```
### 系统信号
操作系统可以向执行中的进程发送信号，`process` 对象能够监听信号事件。

```javascript
process.on('SIGINT', function () {
  console.log('Got a SIGINT');
  process.exit(0);
});
```

发送信号的方法如下。

```
$ kill -s SIGINT [process_id]
```

### 相关库

- [Vorpal](https://github.com/dthree/vorpal) 一个用来构建交互式命令行程序的框架

参考资料：

- http://blog.npmjs.org/post/118810260230/building-a-simple-command-line-tool-with-npm
- http://www.ruanyifeng.com/blog/2015/05/command-line-with-node.html
- https://developer.atlassian.com/blog/2015/11/scripting-with-node/