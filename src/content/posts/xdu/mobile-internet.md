---
title: 西电移动互联网导论 - FileDownloader
published: 2025-09-11
updated: 2025-09-11
image: ''
tags: [西电, 移动互联网导论, Typescript, Node.js]
category: 'Course'
draft: false 
---

# 前言

本程序是移动互联网导论的一次小作业，需要使用 TS+Node.js 实现一个简易下载器。下载器的使用方式如下：

```cmd
filedownloader <url> <outputFile>
```

在项目开始前，请确保你已经安装了 Node.js。

我在本项目中使用的包管理器是 [pnpm](https://pnpm.io/zh/)，如果你还没有安装 `pnpm`，可以使用 `npm`/`npx` 或参考 `pnpm` 官网的[安装指南](https://pnpm.io/zh/installation)进行安装。

想直接看源代码请移步 [Github](https://github.com/Yang-ZhiHang/file-downloader)。

# 开始

## 配置文件

首先一个良好的代码离不开良好的代码组织，我们将源代码部分放在 `src` 目录下，编译后的代码放在 `dist` 目录下（`.ts` 需要编译为 `.js` 进行运行）。

因为项目是基于 TS 的，所以我们需要初始化项目对 `.ts` 文件的配置：

> 如果你是用的是 `npm`，下面的 `pnpx` 改为 `npx`，`pnpm` 改为 `npm`

```cmd
pnpx tsc --init
pnpm add typescript
```

> tsc 即 typescript compiler，是 ts 的编译器。编译为 js 文件后通过解释器环境 node.js 运行。

运行 `pnpx tsc --init` 后会生成一个 `tsconfig.json` 文件，我们需要对其进行一些配置。找到里面的如下部分，取消注释，因为我们需要基于 `node` 环境进行开发：

```json
{
    // File Layout
    "rootDir": "./src",
    "outDir": "./dist",
    // For nodejs:
    "lib": ["esnext"],
    "types": ["node"],
    // and npm install -D @types/node
}
```

另外由于我们项目的主要功能是下载文件，需要和后端打交道，所以我们需要后端模块的相关声明信息：

```cmd
pnpm install @types/node
```

以上，TS 的环境就配置好了。

接下来我们需要配置 `package.json`，由于 `EJS` 是大势所趋，所以我们就不用 `CJS` 了，将 `type` 字段设置为 `module`：

> `EJS` 和 `CJS` 是两种不同的模块化方案，前者是 ES6 标准的模块化方案，后者是 Node.js 早期的模块化方案。可以理解为代码的模块化风格吧~

```json
{
  "type": "module",
}
```

同时，我们需要配置 `scripts` 字段，方便我们编译和运行：

```json
{
  "scripts": {
		"build": "tsc",
		"start": "node dist/main.js",
		"dev": "tsc && node dist/main.js"
	},
}
```

对于刚开始的同学来说，代码先跑起来最重要，后续的功能完善和代码优化可以慢慢来~

在 `main.ts` 里面先写一个简单的 `console.log("Hello, TS")`，然后运行 `pnpm dev`，如果程序输出结果正常，说明环境配置成功。

## 将项目变为可运行的命令行程序

接下来，我们需要将项目变为一个可运行的命令行程序，也就是说无论你在哪个目录下，只要你打开终端命令行，输入 `filedownloader` 都能使用这个程序。

首先，我们需要在 `package.json` 中添加一个 `bin` 字段来指定全局命令行的名称和执行文件入口：

```json
{
  "bin": {
    "filedownloader": "./dist/main.js"
  }
}
```

这个配置指定了最后生成的可执行文件的入口。

最后我们对项目进行编译，并安装为全局命令行工具：

```cmd
pnpm build
pnpm link
```

> `pnpm link` 可能会报错，按照报错提示自己分析并解决问题

完成之后，直接打开 `cmd`，输入 `filedownloader`，如果看到输出 `Hello, TS`，说明配置成功。

接下来咱们就要开始进行模块化编程了~

## 1. 参数输入

我们希望能够按照 `filedownloader www.baidu.com output.txt` 的格式进行输入，那么应该怎么去实现呢？

如果你用的是控制台输入，你会发现你要先在控制台里面输入 `filedownloader`，然后按下回车，接着再输入 `www.baidu.com output.txt`，然后再按下回车。

这样显然不符合我们的需求。

就像 C 语言的 `main` 函数可以接收命令行参数一样，`node` 也提供了类似的功能。

在 `node` 里面，命令行参数可以通过 `process.argv` 来获取。

具体的内容移步 [Node.js 官方文档](https://nodejs.org/api/process.html#processargv)。

> 作为一名程序员，[RTFM](https://en.wikipedia.org/wiki/RTFM) 是非常重要的技能。
>
> 如果你只是想选个水课，结果发现这课太 TM 磨人了，那就尝试问问 AI 吧~

## 2. 参数校验模块

既然我们要实现命令行下载器，那么我们就需要解析命令行参数。

:::note[为什么需要解析命令行参数呢？]
想想，你写的 `filedownloader` 不仅仅是给自己用的，你也希望给别人用（~老师用的时候程序不崩掉~）。
如果用户输入的是：`filedownloader abc ababa`，你就需要告诉用户 `abc` 不是一个合法的 URL。
所以我们需要一个解析模块来帮助我们解析命令行参数。
:::

对于第一个参数 URL，我们可以使用正则表达式来进行简单的验证：

```typescript
const urlPattern = /^(https?:\/\/)?([\w-]+(\.[\w-]+)+)(\/[\w-./?%#&=]*)?$/

if (!urlPattern.test(url)) {
    console.error('Invalid URL format');
    process.exit(1);
}
```

对于第二个参数 output，其实我们只需要确保它是一个合法的文件路径即可。

```typescript
const outputPattern = /^(\.\/)?[\w-]+(\.[\w-]+)?$/;

if (!outputPattern.test(outputPath)) {
    console.error('Invalid output file path');
    process.exit(1);
}
```

接下来是项目最主要的部分：

## 3. 链接下载模块

自己通过代码下载一个文件，Uh..🤔

如果你写过 `AI+前端` 应该会了解到 `fetch` API，毕竟为了实现大模型的流式输出，我们需要使用它来对内容进行流式处理。

回到 `node` 环境，如果你忘了或者不会使用 `fetch`，可以移步 [Node.js 官方文档](https://nodejs.org/en/learn/getting-started/fetch)。

在 `fetch` 官方文档的 `use pool with Undici` 部分，我们可以看到 `fetch` 对文件流的处理方式：

```javascript
const decoder = new TextDecoder();
for await (const chunk of body) {
  partial += decoder.decode(chunk, { stream: true });
  console.log(partial);
}
```

类似地，我们在使用 `fetch` 获取到响应体后，可以通过 `body` 属性获取到一个 `ReadableStream`，然后我们就可以通过流的方式来读取数据。

```javascript
const fileStream = createWriteStream(outputPath);

for await (const chunk of response.body as any as AsyncIterable<Buffer>) {
    fileStream.write(chunk);
    currentSize += chunk.length;
    // ... (更新进度条等操作)
}
fileStream.end();
```

在上面的代码中，我们使用 `fileStream` 对获取到的数据写入到 `outputPath`，中间的异步循环会持续读取数据直到读取完毕。

更新进度条的具体逻辑部分靠你们发挥自己的想象力了，想要怎样的效果自己问 AI 找方案去实现就好了~

# 写在最后

移动互联网导论老师挺好的，上课主要给我们讲大框架的一些认知，具体细节需要课下自学，作业也挺有趣的，能学到东西，不亏~