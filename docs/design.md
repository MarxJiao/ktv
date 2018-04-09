# 设计文档

## 背景

想学习一下基于node的前后端同构的web app开发。

## 设计目标

> **过程最重要**。通过项目搭建提升对后端服务的理解，提升对webapp形态的理解，感受下强类型的语言，向webpack工程师迈进一步。大部分基础环境搭建会参考[vue-cli](https://github.com/vuejs/vue-cli)和[nuxt.js](https://github.com/nuxt/nuxt.js)。

这套环境目标能够让开发人员专注于代码，不用为逻辑之外的事情费心。基本的项目流程如下。ktv会完成开发和编译阶段的工作。

### 开发调试

开发时一键启动，浏览器和node的调试都基于sourcemap。node在本地编辑器调试，推荐使用vscode。前端代码在浏览器调试。

### 编译

前后端都编译，打包在一起。

### 发布

将打包结果放到服务器（通过持续集成工具自动推），一键启动。

## 选型

* `Koa`: 应用比较广泛。没有附加多余的功能，中间件即插即用。
* `Vue`: 上手简单，组件化开发，支持SSR。
* `Typescript`: 为前端开发添加强类型支持。
* `webpack`: 支持中间件，能够在开发时将前端代码打到内存里调试。
* `nodemon`: 开发时监听js文件变化，重启server。
* `jest`: js单元测试框架，能测试前后端代码，配置简单。
* `pm2`/`forever`/`supviser`: 部署时的进程管理。暂时还没想好用哪个。有点忌惮pm2的协议。

## 设计思路

### 总体思路

开发一个前后端同构的webapp。大概会分两个阶段实现。第一阶段先用node 提供api，前端配合service worker搞个pwa出来。这部分重点在于环境搭建，前后端调通和pwa的缓存更新。第二阶段做基于ssr的pwa。重点在首屏怎么缓存，二次打开时数据怎么更新。就是解决后端模板和前端数据之间的矛盾。通过对这个app的开发，最后输出一个叫ktv的开发环境。

### 环境搭建

1. 先搭建个typscript的koa开发环境，不需要过多的环境配置，使用tsc编译koa的源码，node运行编辑结果就行了。开发时使用tsc的watch模式，nodemon监听js文件变换。
2. 搭建基于webpack的Vue开发环境，能编译vue和ts就行。这里需要区分开发环境和打包环境。
3. 在开发时koa使用webpack中间件，直接读取内存中的前端文件。前端代码变更后通过webpack热加载。（每次修改后端代码都会重启webpack，这里可能会有大坑）


## 重点关注

这里列出一些可能会遇到的问题。

1. vscode调试基于typescript的node程序。
2. 开发时启动项目，需要同时进行前后端编译，重启，热加载等。
3. 生产环境部署，项目启动，https支持等。

# 2018.03.08更新

感觉nodemon重启服务，不适合配合koa-webpack使用，因为每次重启都会重新编译前端代码。看了nuxt.js的源码，找到了新的实现思路。nuxt.js前后端都使用webpack编译，开发时，使用webpack-dev-middleware和webpack-hot-middleware实现前端代码的热更新。目前来看是后端代码使用webpack做热加载，加载后并不会立即执行，使用chokidar做加载后的重新执行，使用chokidar，监控了部分文件变化。

使用chokidar监控代码如下：

```javascript
watchFiles() {
    const src = this.options.srcDir
    let patterns = [
        r(src, this.options.dir.layouts),
        r(src, this.options.dir.store),
        r(src, this.options.dir.middleware),
        r(src, `${this.options.dir.layouts}/*.{vue,js}`),
        r(src, `${this.options.dir.layouts}/**/*.{vue,js}`)
    ]
    if (this._nuxtPages) {
        patterns.push(
        r(src, this.options.dir.pages),
        r(src, `${this.options.dir.pages}/*.{vue,js}`),
        r(src, `${this.options.dir.pages}/**/*.{vue,js}`)
        )
    }
    patterns = _.map(patterns, p => upath.normalizeSafe(p))

    const options = Object.assign({}, this.options.watchers.chokidar, {
        ignoreInitial: true
    })
    /* istanbul ignore next */
    const refreshFiles = _.debounce(() => this.generateRoutesAndFiles(), 200)

    // Watch for src Files
    this.filesWatcher = chokidar
        .watch(patterns, options)
        .on('add', refreshFiles)
        .on('unlink', refreshFiles)

    // Watch for custom provided files
    let customPatterns = _.concat(
        this.options.build.watch,
        ..._.values(_.omit(this.options.build.styleResources, ['options']))
    )
    customPatterns = _.map(_.uniq(customPatterns), p =>
        upath.normalizeSafe(p)
    )
    this.customFilesWatcher = chokidar
        .watch(customPatterns, options)
        .on('change', refreshFiles)
}
```