#Crystal
**Crystal** 是一个基于**组件**的快速开发前端框架

![Crystal icon](http://i2.dpfile.com/ba/crystal.jpg)

#目录
- [get start](#get-start)
- [project structure](#project-structure)
- [state](#state)
- [module](#module)
  - [view](#view)
  - [model](#model)
    - [setData(data)](#setdatadata)
    - [getData()](#setdata)
    - [modal(options)](#modaloptions)
    - [fetch(options)](#fetchoptions)
    - [pipe(fns,args..)](#pipefnsargs)
- [page route](#page-route)
- [module layout](#module-layout)
- [redirect](#redirect)
- [service](#service)
  - [notification](#notification)
- [binding-handler](#binding-handler)
  - [module](#bindinghandlermodule)

#get start
- 安装[node](http://nodejs.org/)或者[iojs](https://iojs.org/)
- 安装gulp `npm install gulp -g`
- checkout本项目，以下操作都在本项目目录里进行 
- 安装nodejs依赖 `npm install`
- get start编译 `gulp && gulp watch`
- get startserver，需要另开一个命令行窗口 `node server.js`
- 打开浏览器访问 `http://localhost:3000`，应该能在页面上看到`hello world`

#project structure

```
|-- README.md                      
|-- app                  -- 存放本应用有关代码
|   |-- asset            -- 存放资源文件，包括stylus以及图片
|   |-- binding-handler  -- knockout的binding handlers
|   |-- module           -- 存放组件，这是开发业务逻辑的主要目录
|   `-- service          -- 各个组件需要用到的公用服务
|-- dist/                -- 编译后的文件
|-- gulpfile.js          -- gulp task
|-- index.html           -- get start页面      
|-- node_modules/        -- 存放nodejs的库
|-- package.json         -- nodejs 库的配置
|-- server-config.js     -- 模拟的后台代码
|-- server.js            -- get startserver的脚本
`-- vendor/              -- 非nodejs的库
```

#app state
使用[crystal-state]作为app全局状态

#module
module的编译是由crystal-
module文件是一个html文件，可以包含一个script标签,例如

- 只含html

```html
<div>hello world</div>
```

- 含html和script
```html
<div>hello world</div>
<script>
    model = [
        
    ]
</script>
```

- 也可以只含script
```html
<script>
    model = [
        
    ]
</script>
```
module文件一般存放在app/module目录下
module文件包含
- [view](#view) —— html内容
- [model](#model) —— script标签内容

##view
view基于[knockout]的绑定语法，默认导入了[knockout.punches]的语法。knockout的配置可以查看
`app/setup-knockout.js`

##model
crystal会把model转化成一个[mixin-class]

model可以是一个array（推荐），function，或者object。

如果是数组的话，数组里面的function会被作为构造函数（可以多个，顺序执行）；object会被作为prototype；数组可以嵌套

在`app/vm.js`中定义了所有model的基类，它引入了以下方法：

###setData(data)
data是一个object对象，setData会把data中value生成ko.observable()。
请参考[knockout.mapping]

###getData()
把ko.observable()对象转成普通对象
请参考[knockout.mapping]

###modal(options)
弹出对话框
- options.target: `Object`, modal的class
- options.confirm: `Function`, 当modal调用callback并且传入参数的时候被调用
- options.cancel: `Function`, 当modal调用callback不传入任何参数的时候被调用
- options.compelete: `Function`, 当modal调用callback的时候被调用

请参考`app/service/modal.js`

###fetch(options)
发起ajax请求

用jQuery的ajax实现

请参考`app/service/ajax.js`


###pipe(fns,args..)
把多个异步方法串联起来

- fns: `Array` 需要串联的异步方法
- args..: `Array` 除了fns之外的剩余参数。 在调用fns的第一个方法传入的参数


#page route
使用[crystal-page]作为module文件加载器

当使用state.Location的时候访问 `http://localhost:3000/a/b?c=1&d=2`

或使用state.Hash的时候访问 `http://localhost:3000/any/path/#/a/b?c=1&d=2`

crystal会加载 `app/module/a/b` 

额外的，如果这个module有onStateChange，则这个方法会被调用，并且传入state.getData().query作为参数，例如
``` js
model = [
    {
        onStateChange: function(query) {
            console.log(query);
        }
    }
]
```

#module layout
在module每一层文件夹下面都一个添加一个`__layout.html`文件，该文件会作为对应目录的module layout文件。

__layout文件内必须包含一个`{{content}}`字符串

```html
<main>{{content}}</main>
```
在生成module的时候，{{content}}将会被替换成该目录下module的view

__layout.html可以嵌套在多个目录下

#redirect
在`app/app-config.js`中定义

``` js
module.exports = {
    redirects: {
        '/': '/rotate',

        '/rotate': '/rotate/territory',
        '/rotate/territory': '/rotate/territory/hierarchy',

        '/rotate/team': '/rotate/team/list'
    }
}
```
当state发生变化的时候，crystal查找redirect配置，反复应用，直到最终。

如果当你配置上面那个redirect设置，并且访问 `/` 的时候

页面会进行如下redirect操作
/ > /rotate > /rotate/territor > /rotate/territory/hierarchy

#service
##notification
弹出提示

请参考[toastr]

#binding-handler
##module

定义一个module
```js

    var FormGroup = require('module/control/form-group');
    var TextBox = require('module/control/text-box');
    
    
    this.modules = {
        territoryName: new FormGroup({
            content: new TextBox({
                value: this.territoryName,
                validator: this.validators.territoryName
            }),
            label: '战区名称',
            required: true
        }),
    
        parentTerritory: new FormGroup({
            content: new (require('module/control/territory-typeahead'))({
                value: this.parentTerritoryId,
                text: this.parentTerritoryName,
                validator: this.validators.parentTerritoryId
            }),
            label: '父战区',
            required: true
        })
    }
```

在当前module插入一个module

```html
  <fieldset>
      <legend>
          基本信息
      </legend>
      <!-- ko module: modules.territoryName --><!-- /ko -->
      <!-- ko module: modules.parentTerritory --><!-- /ko -->
  </fieldset>
```


[knockout]: http://www.knockoutjs.com/ 
[browserify]: http://browserify.org/
[crystal-modulify]: https://github.com/youngjay/crystal-modulify
[crystal-page]: https://github.com/youngjay/crystal-page
[crystal-state]: https://github.com/youngjay/crystal-state
[knockout.punches]: http://mbest.github.io/knockout.punches/
[knockout.mapping]: http://knockoutjs.com/documentation/plugins-mapping.html
[mixin-class]: https://github.com/youngjay/mixin-class
[toastr]: http://codeseven.github.io/toastr/demo.html
