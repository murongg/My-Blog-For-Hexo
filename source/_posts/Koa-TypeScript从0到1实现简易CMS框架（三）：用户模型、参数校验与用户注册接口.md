---
title: Koa+TypeScript从0到1实现简易CMS框架（三）：用户模型、参数校验与用户注册接口
thumbnail: https://i.picsum.photos/id/123/565/242.jpg
date: 2020-02-28 08:25:36
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
用户系统是一个cms最重要的部分，也是最复杂的部分，需要进行很多安全处理。  
每次用户请求接口时，我们要进行参数校验，以防用户传入危险以及不规范数据

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

## 初始化Sequelize配置

再`src/core`目录下创建`db.ts`文件，引入`sequelize-typescript`与`config.ts`配置文件。
```javascript
import { Sequelize, Model } from "sequelize-typescript";
import { config, databaseInterface } from "../config/config";
```
### 初始化数据库信息
```javascript
// 数据库配置信息
const { dbName, user, password, host, port }: databaseInterface = config.database;
// 初始化Sequelize
const sequelize: Sequelize = new Sequelize(
  dbName, // 数据库名称
  user, // 数据库用户名
  password, // 数据库密码
  {
    dialect: "mysql", // 数据库引擎
    host, // 数据库地址
    port, // 数据库端口
    logging: true, // 是否打印日志
    timezone: "+08:00", // 设置数据库市区，建议设置，mysql默认的时区比东八区少了八个小时
    define: {
      timestamps: true, // 为模型添加 createdAt 和 updatedAt 两个时间戳字段
      paranoid: true, // 使用逻辑删除。设置为true后，调用 destroy 方法时将不会删队模型，而是设置一个 deletedAt 列。此设置需要 timestamps=true
      underscored: true, // 转换列名的驼峰命名规则为下划线命令规则
      freezeTableName: true // 转换模型名的驼峰命名规则为表名的下划线命令规则
    }
  }
);
```

### 设置`sequelize`是否自动建表
```javascript
sequelize.sync({
  // 是否自动建表
  force: false
});
```
### JSON序列化
JSON序列化是使`sequelize`每次返回都默认排除我们不想要的字段。    `sequelize`的`Model`的原型上会有一个`toJSON`方法，这个是`Model`默认的序列化方法，我们要重写它这个方法：
```javascript
Model.prototype.toJSON = function(): object {
  // 浅拷贝从数据库获取到的数据
  let data = clone(this['dataValues'])
  // 删除指定字段
  unset(data, 'updatedAt')
  unset(data, 'deletedAt')
  // 这个是自己再Model原型上定义的变量
  // 用于控制我们再某次查询数据时想要排除的其他字段
  // 类型为数组，数组的值便是想要排除的字段
  // 例如user.exclude['a', 'b']，此次查询将会增加排除a,b字段
  if(isArray(this['exclude'])) {
    this['exclude'].forEach(value => {
      unset(data, value)
    })
  }
  return data;
};
```
### 全部代码
```javascript
import { Sequelize, Model } from "sequelize-typescript";
import { config, databaseInterface } from "../config/config";
import { unset,clone, isArray } from "lodash";
// 数据库配置信息
const {
  dbName,
  user,
  password,
  host,
  port
}: databaseInterface = config.database;
// 初始化Sequelize
const sequelize: Sequelize = new Sequelize(
  dbName, // 数据库名称
  user, // 数据库用户名
  password, // 数据库密码
  {
    dialect: "mysql", // 数据库引擎
    host, // 数据库地址
    port, // 数据库端口
    logging: true, // 是否打印日志
    timezone: "+08:00", // 设置数据库市区，建议设置，mysql默认的时区比东八区少了八个小时
    define: {
      timestamps: true, // 为模型添加 createdAt 和 updatedAt 两个时间戳字段
      paranoid: true, // 使用逻辑删除。设置为true后，调用 destroy 方法时将不会删队模型，而是设置一个 deletedAt 列。此设置需要 timestamps=true
      underscored: true, // 转换列名的驼峰命名规则为下划线命令规则
      freezeTableName: true // 转换模型名的驼峰命名规则为表名的下划线命令规则
    }
  }
);

sequelize.sync({
  // 是否自动建表
  force: false
});
Model.prototype.toJSON = function(): object {
  // 浅拷贝从数据库获取到的数据
  let data = clone(this['dataValues'])
  // 删除指定字段
  unset(data, 'updatedAt')
  unset(data, 'deletedAt')
  // 这个是自己再Model原型上定义的变量
  // 用于控制我们再某次查询数据时想要排除的其他字段
  // 类型为数组，数组的值便是想要排除的字段
  // 例如user.exclude['a', 'b']，此次查询将会增加排除a,b字段
  if(isArray(this['exclude'])) {
    this['exclude'].forEach(value => {
      unset(data, value)
    })
  }
  return data;
};
export { sequelize };
```
## 创建Users模型

`sequelize-typescript`创建模型和`sequelize`创建模型区别还是挺大的，`sequelize-typescript`中大部分字段的配置都是基于装饰器来实现。下面直接贴上代码，基本看一遍就知道怎么回事了。  

**注意事项：**
- **千万不要忘记@Table装饰器，少写这个装饰器会报错**
- **也不要忘记向Model里传入泛型**

```javascript
import { sequelize } from "../../core/db";
import {
  Model,
  Table,
  Column,
  DataType,
  PrimaryKey,
  AutoIncrement,
  Unique,
  Comment,
} from "sequelize-typescript";
// 千万不要忘记Table装饰器，少写这个装饰器会报错
// 也不要忘记向Model里传入泛型
@Table
class Users extends Model<Users> {
  @PrimaryKey
  @AutoIncrement
  @Comment("ID")
  @Column(DataType.INTEGER)
  id?: number;

  @Comment("用户昵称")
  @Column(DataType.STRING(128))
  nickname?: string;

  @Unique
  @Comment("用户邮箱")
  @Column(DataType.STRING(128))
  email?: string;

  @Comment("用户密码")
  @Column(DataType.STRING(64))
  password?: string;

  @Unique
  @Comment("微信小程序openid")
  @Column(DataType.STRING(128))
  openid?: string;
}

sequelize.addModels([Users]);

export default Users;

```

## 参数校验

> 参数校验是一个系统中必不可少的部分，尤其是前后端分离的架构模式，为了更方便的使用参数校验，我们需要自己封装一个类，实现代码更高的复用性，此类模仿`lin-cms-koa`的参数校验的基本功能进行封装。

### Validator封装
在`src/core`文件夹下创建`validator.ts`文件，引入需要的依赖：
```javascript
import { validateOrReject } from "class-validator";
import { Context } from "koa";
import { cloneDeep } from "lodash";
import { ParametersException } from "./exception";
```
`Validator`类封装思路：
1. 解析koa的`Context`，获取到可能接收到用户传来的参数的字段，进行拍平（扁平化）
2. 遍历所有参数，将它们的`key`挂载到原型上。
3. 使用`class-validator`进行参数校验。

实现代码:
```javascript
export class Validator {
  async validate(ctx: Context) {
    const params = {
      ...ctx.request.body,
      ...ctx.request.query,
      ...ctx.params
    };
    const data = cloneDeep(params);
    for (let key in params) {
      this[key] = params[key];
    }
    try {
      await validateOrReject(this);
      return data;
    } catch (errors) {
      let errorResult: string[] = [];
      errors.forEach(error => {
        let messages: string[] = [];
        for (let msg in error.constraints) {
          messages.push(error.constraints[msg]);
        }
        errorResult = errorResult.concat(messages)
      });
      throw new ParametersException({ msg: errorResult });
    }
  }
}
```
> **具体使用方式在用户注册接口时进行演示**

## 用户注册接口
### 创建`/v1/user/register`路由

在`src/app/api/v1`目录下创建`users.ts`文件，由于我们之前写了路由自动注册功能，所以我们只需要将路由导出即可，不需要再`app.ts`中引入路由。

引入`koa-router`
```javascript
import Router from "koa-router";
const router: Router = new Router();
```
设置路由的`prefix`
```javascript
router.prefix("/v1/user");
```
创建路由：
```javascript
router.post("/register", async ctx => {});
```
### 参数校验
上文我们已经将`Validator`类封装好了，在`src/app/validators`目录下创建`UsersValidator.ts`文件，参数校验是基于`class-validator`，具体使用方式可以观看[官网文档](https://github.com/typestack/class-validator)，直接上基础代码：

```javascript
/**
 * 注册验证类
 *
 * @export
 * @class RegistorValidator
 * @extends {Validator}
 */
export class RegistorValidator extends Validator {
  constructor() {
    super();
  }
  @Length(3, 10, {
    message: "用户名长度为3~10个字符"
  })
  nickname?: string;
  @IsEmail({},{ message: "电子邮箱格式错误" })
  email?: string;
  @Validate(CheckPassword)
  // 至少8-16个字符，至少1个大写字母，1个小写字母和1个数字，其他可以是任意字符：
  @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)[^]{8,16}$/, {
    message: "密码至少8-16个字符，至少1个大写字母，1个小写字母和1个数字"
  })
  password1?: string;
  password2?: string;
}
```
由于我们需要判断`password1`和`password2`是否相等，`class-validator`没有相似功能，我们自己创建一个校验方法：
```javascript
/**
 * 验证密码自定义装饰器
 *
 * @class CheckPassword
 * @implements {ValidatorConstraintInterface}
 */
@ValidatorConstraint()
class CheckPassword implements ValidatorConstraintInterface {
  validate(text: string, args: ValidationArguments): boolean {
    const obj: any = args.object;
    return obj.password1 === obj.password2;
  }
  defaultMessage() {
    return "两次输入密码不一致";
  }
}
```

在`password1`属性上可以直接使用装饰器挂载这个自定义方法：
```javascript
@Validate(CheckPassword)
password1?: string;
```
至此注册接口的校验器完成，全部代码：
```javascript
import {
  Length,
  IsEmail,
  Matches,
  Validate,
  ValidatorConstraintInterface,
  ValidatorConstraint,
  ValidationArguments
} from "class-validator";

import { Validator } from "../../core/validator";

/**
 * 验证密码自定义装饰器
 *
 * @class CheckPassword
 * @implements {ValidatorConstraintInterface}
 */
@ValidatorConstraint()
class CheckPassword implements ValidatorConstraintInterface {
  validate(text: string, args: ValidationArguments): boolean {
    const obj: any = args.object;
    return obj.password1 === obj.password2;
  }
  defaultMessage() {
    return "两次输入密码不一致";
  }
}

/**
 * 注册验证类
 *
 * @export
 * @class RegistorValidator
 * @extends {Validator}
 */
export class RegistorValidator extends Validator {
  constructor() {
    super();
  }
  @Length(3, 10, {
    message: "用户名长度为3~10个字符"
  })
  nickname?: string;
  @IsEmail({},{ message: "电子邮箱格式错误" })
  email?: string;
  @Validate(CheckPassword)
  // 至少8-16个字符，至少1个大写字母，1个小写字母和1个数字，其他可以是任意字符：
  @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)[^]{8,16}$/, {
    message: "密码至少8-16个字符，至少1个大写字母，1个小写字母和1个数字"
  })
  password1?: string;
  password2?: string;
}
```
在路由文件中使用校验器，调用校验器类上的`validate`方法，将`koa`的`Context`传入。  
如果校验成功则将请求参数封装成一个对象并返回，  
如果失败则直接利用全局异常处理中间件 向客户抛出错误信息：
```javascript
router.post("/register", async ctx => {
  const v: registerInterface = await new RegistorValidator().validate(ctx);
});
```
`registerInterface`接口存放了注册所需要的参数，代码：
```javascript
export interface registerInterface {
  email: string;
  nickname: string;
  password1: string;
  password2: string;
}
```
### 实现注册功能
在`src/app/service`目录下创建`users.ts`文件，此目录专门存放进行数据库业务操作的文件。

在`users.ts`中创建`UsersService`类，在类中创建静态方法`userRegister`，此方法进行注册操作。

**注册步骤：**
1. 判断数据库中是否存在此用户
2. 如果存在则向用户抛出异常
3. 如果不存在则将数据插入数据库

业务代码：
```javascript
static async userRegister(params: registerInterface) {
  const { email, nickname, password1 } = params;
  const data = {
    email,
    nickname,
    password: password1
  };
  const isExistEmail = await Users.findOne({
    where: {
      email
    }
  });
  if (isExistEmail) {
    throw new Failed({ msg: "Email已存在" });
  }
  const r = await Users.create(data);
  return r;
}
```

全部代码：
```javascript
import Users from "../models/users";
import { Failed } from "../../core/exception";
import { registerInterface } from "../lib/interface/UsersInterface";
class UsersService {
  static async userRegister(params: registerInterface) {
    const { email, nickname, password1 } = params;
    const data = {
      email,
      nickname,
      password: password1
    };
    const isExistEmail = await Users.findOne({
      where: {
        email
      }
    });
    if (isExistEmail) {
      throw new Failed({ msg: "Email已存在" });
    }
    const r = await Users.create(data);
    return r;
  }
}
export default UsersService;
```
在路由中引入注册功能代码：
```javascript
router.post("/register", async ctx => {
  const v: registerInterface = await new RegistorValidator().validate(ctx);
  const r = await UsersService.userRegister(v);
  if (r) {
    throw new Success();
  } else {
    throw new Failed({msg: '注册失败'});
  }
});
```
### 路由文件全部代码：
```javascript
import Router from "koa-router";
import { RegistorValidator } from "../../validators/UsersValidator";
import { Success, Failed } from "../../../core/exception";
import { registerInterface } from '../../lib/interface/UsersInterface';
import UsersService from '../../service/users';

const router: Router = new Router();
router.prefix("/v1/user");

router.post("/register", async ctx => {
  const v: registerInterface = await new RegistorValidator().validate(ctx);
  const r = await UsersService.userRegister(v);
  if (r) {
    throw new Success();
  } else {
    throw new Failed({msg: '注册失败'});
  }
});
// 这里一定要用commonjs规范导出
module.exports = router;
```
## 更新中......