# KTV

A web app project base on Koa, Typescript and Vue.

## 调试

开发调试使用nodemon来自动重启node进程，所以需要全局安装nodemon。

```shell
npm i -g nodemon
```

### 使用vscode

推荐使用vscode进行开发。在调试面板执行"Start and Debug"。程序会自动编译，启动。启动后用浏览器打开[http://localhost:3000](http://localhost:3000)查看页面。修改后端代码会自动重新编译和重启服务。

### 使用npm 命令。

1. 先打开typescript编译

```shell
npm run tsc:watch
```

或者使用yarn

```shell
yarn tsc:watch
```

2. 使用nodemon执行编译后的结果

```shell
npm run dev
```

或者使用yarn

```shell
yarn dev
```

3. 打开页面

启动后用浏览器打开[http://localhost:3000](http://localhost:3000)查看页面。
