# CMA项目文档

测试地址：[https://testing.hulacorp.com/lifetouch-admin/](https://testing.hulacorp.com/lifetouch-admin/)

OP端测试地址：[http://testing.hulacorp.com/lifetouch-opra/](http://testing.hulacorp.com/lifetouch-opra/)

测试账号：

账号：15078426926 密码：123456

账号：18077543779 密码：a123456（Lydia账号）

线上地址：[https://hulacorp.com/lifetouch-admin/](https://hulacorp.com/lifetouch-admin/)

OP线上地址：[https://hulacorp.com/lifetouch-opra/](https://hulacorp.com/lifetouch-opra/)

线上测试账号：

账号：15078426926 密码：123456

## 接口说明

API 接口分为`CMA` `FPE` ~~`STORE`~~ `REPORT` `JQB_NODE`服务器，见`src/apis/base/constant.js`

1. `CMA` - 主要 API 服务器，[接口文档地址](http://testing.hulacorp.com/lifetouch-cma-api/swagger.jsp?baseUrl=http%3A%2F%2Ftesting.hulacorp.com%2Flifetouch-cma-api%2Fswagger%2Fjson%2Fcommunity%2Fswagger.json%3Ft%3D1587459964218)
2. `FPE` - 文档导入导出打印等接口所在服务器，[接口文档地址](http://testing.hulacorp.com/lifetouch-cma-api/swagger.jsp?baseUrl=http%3A%2F%2Ftesting.hulacorp.com%2Flifetouch-fpe-api%2Fswagger%2Fjson%2Fcommunity%2Fswagger.json%3Ft%3D1587459964218)
3. `STORE`
4. `REPORT`
5. `JQB_NODE`

## 命令说明

1. `serve`：运行本地测试 *
2. `build`: 正式打包至生产环境
3. `analysis`: 项目分析，生成 stats.json 文件
5. `deploy:testing`: 自动打包并部署到 testing 测试环境服务器 *

## 开发说明

### 项目结构
开发中常用目录：
```
/src
  /apis                     // api文件目录
  /assets/i18n              // i18n国际语言包
  /components               // 常用组件
  /enum                     // 页面枚举变量统一存放目录
  /fun-components           // 组件
  /mixins                   // mixin
  /router                   // 路由
  /services                 // service 层
  /store                    // vuex store
  /styles                   // 通用样式
    /svg                    // svg文件
  /utils                    // 工具类
  /views                    // 视图层
```

项目结构先分层，后分模块，`apis` `services` 原为一级扁平结构，分模块的`apis` `services`是本人接手后创建。

* `*.api.js` 文件无需手动引入`http` `CONTANT` `createApi` 模块，已经在`webpack`配置中自动引入。

### API

`api`层方案（三种）：

1. 原有`api`文件采用`http`模块，每个`api`对应一个函数。
2. `createApi`是基于`http`模块进行的封装，实现配置式`api`声明，`/apis/frontdesk/setting/sugset.api.js`中采用。
3. `nextApi`是直接基于`axios`重新进行封装，不依赖`http`实现的配置声明式`api`（需要手动引入）

字段映射

* 项目实际开发中前端字段与后端字段经常不一致，所以可以用`convert`工具进行字段映射，`createApi`没有包含映射，`nextApi`可以直接配置入参和出参映射，直接返回映射后的结果。
* 字段映射工具见：`/utis/DataConverter.js`，已封装成`@converter`装饰器，可直接在`services`层使用（如果使用`nextApi`可以直接在`api`层配置，不再使用装饰器）

### Service

`services`层

* 引入对应`api`依赖，调用`api`并对数据进行处理。
* `mapServices` 工具函数，使用方法同 `mapAction` 等，见`/services/utils/xService.js`，可以减少`vuex store`模块的使用，部分接口可以直接在`view`中调用`service`。

### 页面开发

1. i18n 文件`route.module.json`中添加新页面名称
2. `/enum/types/permissionEnums.js`添加页面权限（值由后端决定）
3. `/enum/types/routeEnums.js`添加页面组件名
4. 对应路径下创建新页面文件
5. 修改路由文件新增路由（`meta`中`permission`控制权限）
6. 修改`/src/views/layout/Main.vue`添加白名单（旧页面样式有`padding`，新页面需要添加白名单应用新样式）

### 权限控制

* 页面权限在路由文件的`meta/permission`字段配置，如果只配置`PERMISSION_PAGE.XXX`，则默认根据`PERMISSION_ACTION.VIEW`判定是否显示当前页面，如果需要根据其他权限字段判定可以指定`permission`为函数。
* 页面内权限控制可以通过`this.$permission(PERMISSION_PAGE.XXX, PERMISSION_ACTION.XXX)`（`mixins/index.js`文件中注入）判定。
* 也可以通过`@permission(PERMISSION_PAGE.XXX, PERMISSION_ACTION.XXX)`装饰器，权限校验通过执行函数，校验失败自动提示权限不足。

### 装饰器
见`/decorator.js`
- `@loading()` 显示loading圈圈
- `@permission()` 校验权限
- `@confirm(args:string|function|object)` 弹出确认提示框，确认后执行函数，取消则不执行
- `@validate(ref:string)` 校验表单，校验通过执行函数，提供`form`组件的`ref`

### 推荐 VSCode 插件

1. [lifetouch-cma-cli](vscode:extension/yindongfang.lifetouch-cma-cli)

路由查看文件定位，SVG文件预览等功能

2. [i18n Ally](vscode:extension/antfu.i18n-ally) i18n 本地化插件

推荐配置，支持识别路由中的`title`

```json
{
  "i18n-ally.sourceLanguage": "zh-CN",
  "i18n-ally.regex.usageMatchAppend": [
    "meta:\\s*\\{[^\\}]*title:\\s*\\(?['\\\"`]({key})['\\\"`]\\)?"
  ]
}
```

3. [TODO Highlight](vscode:extension/wayou.vscode-todo-highlight) 提示高亮插件

推荐配置，添加`vue`文件支持

```json
{
  "todohighlight.include": [
    "**/*.vue",
    "**/*.js",
    "**/*.jsx",
    "**/*.ts",
    "**/*.tsx",
    "**/*.html",
    "**/*.css",
    "**/*.scss"
  ]
}
```
