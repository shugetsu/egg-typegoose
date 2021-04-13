# egg-typegoose

[![NPM version][npm-image]][npm-url]
[![build status][travis-image]][travis-url]
[![Test coverage][codecov-image]][codecov-url]
[![David deps][david-image]][david-url]
[![Known Vulnerabilities][snyk-image]][snyk-url]
[![npm download][download-image]][download-url]

[npm-image]: https://img.shields.io/npm/v/egg-typegoose.svg?style=flat-square
[npm-url]: https://npmjs.org/package/egg-typegoose
[travis-image]: https://img.shields.io/travis/eggjs/egg-typegoose.svg?style=flat-square
[travis-url]: https://travis-ci.org/eggjs/egg-typegoose
[codecov-image]: https://img.shields.io/codecov/c/github/eggjs/egg-typegoose.svg?style=flat-square
[codecov-url]: https://codecov.io/github/eggjs/egg-typegoose?branch=master
[david-image]: https://img.shields.io/david/eggjs/egg-typegoose.svg?style=flat-square
[david-url]: https://david-dm.org/eggjs/egg-typegoose
[snyk-image]: https://snyk.io/test/npm/egg-typegoose/badge.svg?style=flat-square
[snyk-url]: https://snyk.io/test/npm/egg-typegoose
[download-image]: https://img.shields.io/npm/dm/egg-typegoose.svg?style=flat-square
[download-url]: https://npmjs.org/package/egg-typegoose

<!--
Description here.
-->

## Install

```bash
$ npm i egg-typegoose --save
```

## Usage

```js
// {app_root}/config/plugin.ts
exports.typegoose = {
  enable: true,
  package: 'egg-typegoose',
};
```

## Configuration

```js
// {app_root}/config/config.default.ts
exports.typegoose = { 
  url: 'mongodb://localhost:27017/test',
  options: {},
  modelWhitelist: ['BaseModel'], // 文件白名单列表, 不挂载到ctx.model
};
```

see [config/config.default.ts](config/config.default.ts) for more detail.

## Example

```js
├── controller
│    └── user.ts
├── model
│    ├── BaseModel.ts
│    └── User.ts
├── service
│    └── user.ts
├── router.ts
```

```js
// {app_root}/app/controller/user.ts
import { Controller } from 'egg';
import User from '../model/User';

/**
 * User Controller
 */
export default class UserController extends Controller {
  /**
   * 创建用户
   */
  public async createUser() {
    const { ctx } = this;
    const user = new User();
    user.email = 'test@qq.com';
    user.nickname = 'YuYin';
    const result = await ctx.service.user.createUser(user);
    ctx.body = result;
  }

  /**
   * 根据 email 获取用户
   */
  public async getUserByEmail() {
    const { ctx } = this;
    const email = 'test@qq.com';
    const result = await ctx.service.user.getUserByEmail(email);
    ctx.body = result;
  }
}
```

```js
// {app_root}/app/model/BaseModel.ts
import { prop, pre } from '@typegoose/typegoose';

@pre<BaseModel>('save', function(next) {
  if (!this.createdAt || this.isNew) {
    this.createdAt = this.updatedAt = new Date();
  } else {
    this.updatedAt = new Date();
  }
  next();
})

/**
 * Base Model
 */
class BaseModel {
  /**
   * 创建时间
   */
  @prop()
  createdAt: Date;

  /**
   * 更新时间
   */
  @prop()
  updatedAt: Date;
}

export default BaseModel;
```

```js
// {app_root}/app/model/User.ts
import { prop } from '@typegoose/typegoose';
import BaseModel from './BaseModel';

/**
 * User Model
 */
export class User extends BaseModel {
  /**
   * 邮箱
   */
  @prop({ required: true, unique: true })
  email!: string;

  /**
   * 昵称
   */
  @prop()
  nickname?: string;
}

export default User;
```

```js
// {app_root}/app/service/User.ts
import { Service } from 'egg';
import User from '../model/User';

/**
 * User Service
 */
export default class UserService extends Service {
  /**
   * 创建用户
   * @param user User
   */
  public async createUser(user: User) {
    const { ctx } = this;
    const result = await ctx.model.User.create(user);
    return result;
  }
  
  /**
   * 根据 email 获取用户
   * @param email 邮箱
   */
  public async getUserByEmail(email: string) {
    const { ctx } = this;
    const result = await ctx.model.User.findOne({ email });
    return result;
  }
}
```

```js
// {app_root}/app/router.ts
import { Application } from 'egg';

export default (app: Application) => {
  const { controller, router } = app;

  router.post('/user/createUser', controller.user.createUser);
  router.get('/user/getUserByEmail', controller.user.getUserByEmail);
};
```

## Questions & Suggestions

Please open an issue [here](https://github.com/eggjs/egg/issues).

## License

[MIT](LICENSE)
