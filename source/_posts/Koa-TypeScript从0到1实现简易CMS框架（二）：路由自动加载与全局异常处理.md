---
title: Koa+TypeScript从0到1实现简易CMS框架（二）：路由自动加载与全局异常处理
thumbnail: https://i.picsum.photos/id/178/565/242.jpg
date: 2020-02-28 08:25:27
categories:
  - 程序世界
  - WEB后端
  - NodeJs
  - TypeScript
tags:
  - Koa
  - cms
  - 后端
  - RESTFUL
---

 ## 目录
- [Koa+TypeScript从0到1实现简易CMS框架（一）：项目搭建以及配置](/2020/02/28/Koa-TypeScript从0到1实现简易CMS框架（一）：项目搭建以及配置/)  
- [Koa+TypeScript从0到1实现简易CMS框架（二）：路由自动加载与全局异常处理](/2020/02/28/Koa-TypeScript从0到1实现简易CMS框架（二）：路由自动加载与全局异常处理/)
- [Koa+TypeScript从0到1实现简易CMS框架（三）：用户模型、参数校验与用户注册接口](/2020/02/28/Koa-TypeScript从0到1实现简易CMS框架（三）：用户模型、参数校验与用户注册接口/)  


项目地址：[koa-typescript-cms](https://github.com/MuRongXiaoDouBi/koa-typescript-cms) 
## 前言
`koa`本身是没有路由的，需借助第三方库`koa-router`实现路由功能，但是路由的拆分，导致`app.ts`里需要引入许多路由文件，为了方便，我们可以做一个简单的路由自动加载功能来简化我们的代码量；全局异常处理是每个cms框架中比不可少的部分，我们可以通过`koa`的中间件机制来实现此功能。

## 主要工具库
- **koa**   web框架
- **koa-bodyparser** 处理koa post请求
- **koa-router** koa路由
- **sequelize、sequelize-typescript、mysql2** ORM框架与Mysql
- **validator、class-validator** 参数校验
- **jsonwebtoken** jwt
- **bcryptjs** 加密工具
- **reflect-metadata** 给装饰器添加各种信息
- **nodemon** 监听文件改变自动重启服务
- **lodash** 非常好用的工具函数库

<!-- more -->
## 项目目录
```
├── dist                                        // ts编译后的文件
├── src                                         // 源码目录
│   ├── components                              // 组件
│   │   ├── app                                 // 项目业务代码
│   │   │   ├── api                             // api层
│   │   │   ├── service                         // service层
│   │   │   ├── model                           // model层
│   │   │   ├── validators                      // 参数校验类
│   │   │   ├── lib                             // interface与enum
│   │   ├── core                                // 项目核心代码
│   │   ├── middlewares                         // 中间件
│   │   ├── config                              // 全局配置文件
│   │   ├── app.ts                              // 项目入口文件
├── tests                                       // 单元测试
├── package.json                                // package.json                                
├── tsconfig.json                               // ts配置文件
```
## 路由自动加载
思路:（此功能借鉴lin-cms开源的lin-cms-koa-core）
1. 获取`api`文件夹下的所有文件
2. 判断文件的后缀名是否为`.ts`，如果是，使用`CommonJS`规范加载文件
3. 判断文件导出的内容是否为`Router`类型，如果是，则加载路由

由于我们需要很多功能到要在服务执行后就加载，所以创建一个专门加载功能的类`InitManager`。  
再`InitManager`类中创建类方法`initLoadRouters`，此方法专门作为加载路由的功能模块。  
先创建一个辅助函数`getFiles`，此函数利用`node`的`fs`文件功能模块，来获取某文件夹下后的所有文件名，并返回一个字符串数组：
```javascript
/**
 * 获取文件夹下所有文件名
 *
 * @export
 * @param {string} dir
 * @returns
 */
export function getFiles(dir: string): string[] {
  let res: string[] = [];
  const files = fs.readdirSync(dir);
  for (const file of files) {
    const name = dir + "/" + file;
    if (fs.statSync(name).isDirectory()) {
      const tmp = getFiles(name);
      res = res.concat(tmp);
    } else {
      res.push(name);
    }
  }
  return res;
}
```
接下来编写路由自动加载功能：
```javascript
  /**
   * 路由自动加载
   *
   * @static
   * @memberof InitManager
   */
  static initLoadRouters() {
    const mainRouter = new Router();
    const path: string = `${process.cwd()}/src/app/api`;
    const files: string[] = getFiles(path);
    for (let file of files) {
      // 获取文件后缀名
      const extention: string = file.substring(
        file.lastIndexOf("."),
        file.length
      );
      if (extention === ".ts") {
        // 加载api文件夹下所有文件
        // 并检测文件是否是koa的路由
        // 如果是路由便将路由加载
        const mod: Router = require(file);
        if (mod instanceof Router) {
          // consola.info(`loading a router instance from file: ${file}`);
          get(mod, "stack", []).forEach((ly: Router.Layer) => {
            consola.info(`loading a route: ${get(ly, "path")}`);
          });
          mainRouter.use(mod.routes()).use(mod.allowedMethods());
        }
      }
    }
  }
```
在`InitManager`中创建另一个类方法`initCore`，此方法需传入一个`koa`实例，统一加载`InitManager`类中的其他功能模块。  
```javascript
  /**
   * 入口方法
   *
   * @static
   * @param {Koa} app
   * @memberof InitManager
   */
  static initCore(app: Koa) {
    InitManager.app = app;
    InitManager.initLoadRouters();
  }
```
> 需要注意的是，路由文件导出的时候不能再以`ES`的规范导出了，必须以`CommonJS`的规范进行导出。  

例`api/v1/book.ts`文件源码：
```javascript
import Router from 'koa-router'

const router: Router = new Router();
router.prefix('/v1/book')
router.get('/', async (ctx) => {
    ctx.body = 'Hello Book';
});

// 注意这里
module.exports = router 
```
最后在`app.ts`中加载，代码：
```javascript
import InitManager from './core/init'

InitManager.initCore(app)
```
此为还需要全局加载配置文件，与加载路由大同小异，代码一并附上  

`app/core/init.ts`全部代码：
```javascript
import Koa from "koa";
import Router from "koa-router";
import consola from "consola";
import { get } from "lodash";
[
import { getFiles } from "./utils";
import { config, configInterface } from "../config/config";
declare global {
  namespace NodeJS {
    interface Global {
      config?: configInterface;
    }
  }
}
class InitManager {
  static app: Koa<Koa.DefaultState, Koa.DefaultContext>;

  /**
   * 入口方法
   *
   * @static
   * @param {Koa} app
   * @memberof InitManager
   */
  static initCore(app: Koa) {
    InitManager.app = app;
    InitManager.initLoadRouters();
    InitManager.loadConfig();
  }

  /**
   * 路由自动加载
   *
   * @static
   * @memberof InitManager
   */
  static initLoadRouters() {
    const mainRouter = new Router();
    const path: string = `${process.cwd()}/src/app/api`;
    const files: string[] = getFiles(path);
    for (let file of files) {
      // 获取文件后缀名
      const extention: string = file.substring(
        file.lastIndexOf("."),
        file.length
      );
      if (extention === ".ts") {
        // 加载api文件夹下所有文件
        // 并检测文件是否是koa的路由
        // 如果是路由便将路由加载
        const mod: Router = require(file);
        if (mod instanceof Router) {
          // consola.info(`loading](https://note.youdao.com/) a router instance from file: ${file}`);
          get(mod, "stack", []).forEach((ly: Router.Layer) => {
            consola.info(`loading a route: ${get(ly, "path")}`);
          });
          mainRouter.use(mod.routes()).use(mod.allowedMethods());
        }
      }
    }
  }

  /**
   * 载入配置文件
   *
   * @static
   * @memberof InitManager
   */
  static loadConfig() {
    global.config = config;
  }
}
export default InitManager;

```
## 全局异常处理

此功能需依赖`koa`的中间件机制进行开发  
异常分为已知异常与未知异常，需针对其进行不同处理

常见的已知异常：路由参数错误、从数据库查询查询到空数据……  
常见的未知错误：不正确的代码导致的依赖库报错……

已知异常我们需要向用户抛出，以`json`的格式返回到客户端。  
而未知异常一般只有在开发环境才会让它抛出，并且只有开发人员可以看到。

已知异常向用户抛出时，需携带错误信息、错误代码、请求路径等信息。  
我们需要针对已知异常封装一个类，用来标识错误为已知异常。  
在`app/core`目录下创建文件`exception.ts`，此文件里有一个基类`HttpException`，此类继承`JavaScript`的内置对象`Error`，之后所有的已知异常类都将继承`HttpException`。  
代码：
```javascript
/**
 * HttpException 类构造函数的参数接口
 */
export interface Exception {
  code?: number;
  msg?: any;
  errorCode?: number;
}
export class HttpException extends Error {
  /**
   * http 状态码
   */
  public code: number = 500;

  /**
   * 返回的信息内容
   */
  public msg: any = "服务器未知错误";

  /**
   * 特定的错误码
   */
  public errorCode: number = 999;

  public fields: string[] = ["msg", "errorCode"];

  /**
   * 构造函数
   * @param ex 可选参数，通过{}的形式传入
   */
  constructor(ex?: Exception) {
    super();
    if (ex && ex.code) {
      assert(isInteger(ex.code));
      this.code = ex.code;
    }
    if (ex && ex.msg) {
      this.msg = ex.msg;
    }
    if (ex && ex.errorCode) {
      assert(isInteger(ex.errorCode));
      this.errorCode = ex.errorCode;
    }
  }
}
```

针对以上的情况进行编码`app/middlewares/exception.ts`全部代码：
```javascript
import { BaseContext, Next } from "koa";
import { HttpException, Exception } from "../core/exception";
interface CatchError extends Exception {
  request?: string;
}
const catchError = async (ctx: BaseContext, next: Next) => {
  try {
    await next();
  } catch (error) {
    const isHttpException = error instanceof HttpException
    const isDev = global.config?.environment === "dev"
    
    if (isDev && !isHttpException) {
      throw error;  
    }
    if (isHttpException) {
      const errorObj: CatchError = {
        msg: error.msg,
        errorCode: error.errorCode,
        request: `${ctx.method} ${ctx.path}`
      };
      ctx.body = errorObj;
      ctx.status = error.code;
    } else {
      const errorOjb: CatchError = {
        msg: "出现异常",
        errorCode: 999,
        request: `${ctx.method} ${ctx.path}`
      };
      ctx.body = errorOjb;
      ctx.status = 500;
    }
  }
};

export default catchError;
```
最后，`app.ts`里使用中间件，`app.ts`代码：
```javascript
import Koa from 'koa';
import InitManager from './core/init'
import catchError from './middlewares/exception';

const app = new Koa()
app.use(catchError)
InitManager.initCore(app)

app.listen(3001);

console.log('Server running on port 3001');
```

**下一篇：[Koa+TypeScript从0到1实现简易CMS框架（三）：用户模型、参数校验与用户注册接口](/2020/02/28/Koa-TypeScript从0到1实现简易CMS框架（三）：用户模型、参数校验与用户注册接口/)**