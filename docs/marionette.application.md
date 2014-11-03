# Marionette.Application

`Backbone.Marionette.Application` 对象是复合应用程序的枢纽.它组织，初始化并且协调您的应用程序.如果你喜欢，它为你提供了一个节点，在你的html>script标签或者javascript文件里.

`Application` 是这样直接初始化实例,你也可以扩展它来添加自己的功能.

```js
MyApp = new Backbone.Marionette.Application();
```

## 目录索引

* [添加初始化器](#adding-initializers)
* [应用程序事件](#application-event)
* [启动应用程序](#starting-an-application)
* [系统消息](#messaging-systems)
  * [事件聚合](#event-aggregator)
  * [请求响应](#request-response)
  * [命令](#commands)
* [区域和应用程序对象](#regions-and-the-application-object)
  * [jQuery选择](#jquery-selector)
  * [自定义区域类型](#custom-region-type)
  * [自定义区域类型选择](#custom-region-type-and-selector)
  * [按名称获取区域](#get-region-by-name)
  * [删除区域](#removing-regions)

## 添加初始化器

你的应用程序需要做有用的事情, 像在区域里面显示内容,启动你的路由并且其他事情. 为了完成这些并且确保你的 `Application` 配置成功，你可以添加初始化并且回掉到你的应用程序.

```js
MyApp.addInitializer(function(options){
  // do useful stuff here
  var myView = new MyView({
    model: options.someModel
  });
  MyApp.mainRegion.show(myView);
});

MyApp.addInitializer(function(options){
  new MyAppRouter();
  Backbone.history.start();
});
```

当你启动应用程序，这些回调将被执行，并且被绑定到应用程序对象作为上下文回调.换句话说，`this`是`MyApp`对象，初始化函数的内部.

`options`参数是通过`start`传递的（见下文）.

无论何时你将它们添加到应用程序对象,初始化的回掉都可以被执行. 如果你添加她们在程序开始前，它们就会在`start`函数前执行.如果你添加在程序执行之后，它们会立刻执行.

## 应用程序事件

在`Application`对象的生命周期中引发一些事件,使用[Marionette.triggerMethod](./marionette.functions.md)该功能.
这些事件可以用来做你的应用程序的额外的处理.举个例子, 你可能想预先处理一些数据的初始化发生之前.或者，您可能要等到整个应用程序初始化启动`Backbone.history`.

下面这些事件都会触发:

* **"initialize:before" / `onInitializeBefore`**: fired just before the initializers kick off
* **"initialize:after" / `onInitializeAfter`**: fires just after the initializers have finished
* **"start" / `onStart`**: fires after all initializers and after the initializer events

```js
MyApp.on("initialize:before", function(options){
  options.moreData = "Yo dawg, I heard you like options so I put some options in your options!"
});

MyApp.on("initialize:after", function(options){
  if (Backbone.history){
    Backbone.history.start();
  }
});
```

`options`参数是通过`start`传递的（见下文）.

## 启动应用程序

第一次配置你的应用程序，你可以通过调用`MyApp.start(options)`触发时间.

这个函数接受一个可选的参数. 这个参数可以通过每一个初始化的函数以及初始化事件.这使您可以为您的应用程序的各个部分提供额外的配置, 在应用程序的初始化/启动, 而不是仅仅在定义.

```js
var options = {
  something: "some value",
  another: "#some-selector"
};

MyApp.start(options);
```

## 系统消息

应用实例有三个例子 [messaging systems](http://en.wikipedia.org/wiki/Message_passing) 关于 `Backbone.Wreqr` 看链接. 本节将给出系统的简要概述; 为更深入地了解我们鼓励你阅读 [`Backbone.Wreqr` documentation](https://github.com/marionettejs/backbone.wreqr).

### 事件聚合

事件聚合通过 `vent`属性. `vent` 方便您的应用程序块之间共享信息事件.

```js
MyApp = new Backbone.Marionette.Application();

// 提醒用户"minutePassed"事件
MyApp.vent.on("minutePassed", function(someData){
  alert("Received", someData);
});

// 这将每分钟发出一个事件"minutePassed"
window.setInterval(function() {
  MyApp.vent.trigger("minutePassed", window.someData);
}, 1000 * 60);
```

### 请求响应

Request Response is a means for any component to request information from another component without being tightly coupled. An instance of Request Response is available on the Application as the `reqres` property.

```js
MyApp = new Backbone.Marionette.Application();

// 建立一个处理程序根据类型返回todolist的
MyApp.reqres.setHandler("todoList", function(type){
  return this.todoLists[type];
});

// 获得购物清单要求
var groceryList = MyApp.reqres.request("todoList", "groceries");

// 请求的方法，也可直接从应用程序对象访问
var groceryList = MyApp.request("todoList", "groceries");
```

### 命令

命令用于使任何组件告诉另一个组件不直接引用它执行的操作. 在应用中`commands`属性下的命令实例可用.

注意，一个命令的回调并不意味着有返回值.

```js
MyApp = new Backbone.Marionette.Application();

MyApp.model = new Backbone.Model();

// 建立处理程序获取数据
MyApp.commands.setHandler("fetchData", function(reset){
  MyApp.model.fetch({reset: reset});
});

// 获取数据
MyApp.commands.execute("fetchData", true);

// 可以直接从应用程序执行此功能
MyApp.execute("fetchData", true);
```

## 区域和应用程序对象

Marionette's `Region` 对象，可以通过调用`addRegions`方法直接加入到一个应用程序.

有三种方式，将一个区域变成一个应用程序对象.

### jQuery 选择器

先是要使用一个jQuery选择器选作区域定义的值.这将直接创建Marionette.Region的一个实例,并将其分配给选择:

```js
MyApp.addRegions({
  someRegion: "#some-div",
  anotherRegion: "#another-div"
});
```

### 自定义区域类型

第二个是指定一个自定义区域类型, 其中，区域类型已经指定一个选择器:

```js
MyCustomRegion = Marionette.Region.extend({
  el: "#foo"
});

MyApp.addRegions({
  someRegion: MyCustomRegion
});
```

### 自定义区域类型选择

第三种方法是指定自定义区域类型和一个jQuery选择器，用于该区域的实例,使用对象:

```js
MyCustomRegion = Marionette.Region.extend({});

MyApp.addRegions({

  someRegion: {
    selector: "#foo",
    regionType: MyCustomRegion
  },

  anotherRegion: {
    selector: "#bar",
    regionType: MyCustomRegion
  }

});
```

### 通过名称获取区域

一个区域可以通过名称搜索，使用`getRegion`方法:

```js
var app = new Marionette.Application();
app.addRegions({ r1: "#region1" });

// r1 === r1Again; true
var r1 = app.getRegion("r1");
var r1Again = app.r1;
```

通过指定属性访问的区域相当于从`getRegion`方法访问它.

### 删除区域

Regions可以用'removeRegion`方法去除, 通过一个区域的名称删除（字符串）:

```js
MyApp.removeRegion('someRegion');
```

从应用程序删除区域之前会正确关闭自己.

有关区域的详细信息, 看[the region documentation](./marionette.region.md)
