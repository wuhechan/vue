# **Vue 源码解读**

## Vue 构造器

eg

```
let vm = new Vue({
    el: '#app',
    data: {
        message: '数据'
    }
})
```
code
```
(function (global, factory) {
  // 遵循UMD规范
  typeof exports === 'object' && typeof module !== 'undefined' ? module.exports = factory() :
  typeof define === 'function' && define.amd ? define(factory) :
  (global = global || self, global.Vue = factory());
}(this, function () { 'use strict';
  ···
  // Vue 构造函数
  function Vue (options) {
    // 保证了无法直接通过Vue()去调用，只能通过new的方式去创建实例
    if (!(this instanceof Vue)
    ) {
      warn('Vue is a constructor and should be called with the `new` keyword');
    }
    this._init(options);
  }
  return Vue
})
```
instanceof 运算符是用来检测某个实例对象的原型链上是否存在构造函数的 prototype 属性
## 定义原型属性方法
```
// 定义Vue原型上的init方法(内部方法)
initMixin(Vue);
// 定义原型上跟数据相关的属性方法
stateMixin(Vue);
//定义原型上跟事件相关的属性方法
eventsMixin(Vue);
// 定义原型上跟生命周期相关的方法
lifecycleMixin(Vue);
// 定义渲染相关的函数
renderMixin(Vue);
```
首先initMixin定义了内部在实例化Vue时会执行的初始化代码，它是一个内部使用的方法。
```
function initMixin (Vue) {
  Vue.prototype._init = function (options) {}
}
```
stateMixin方法会定义跟数据相关的属性方法，例如代理数据的访问，我们可以在实例上通过this.$data和this.$props访问到data,props的值，并且也定义了使用频率较高的this.$set,this.$delte等方法。
```
function stateMixin (Vue) {
    var dataDef = {};
    dataDef.get = function () { return this._data };
    var propsDef = {};
    propsDef.get = function () { return this._props };
    {
      dataDef.set = function () {
        warn(
          'Avoid replacing instance root $data. ' +
          'Use nested data properties instead.',
          this
        );
      };
      propsDef.set = function () {
        warn("$props is readonly.", this);
      };
    }
    // 代理了_data,_props的访问
    Object.defineProperty(Vue.prototype, '$data', dataDef);
    Object.defineProperty(Vue.prototype, '$props', propsDef);
    // $set, $del
    Vue.prototype.$set = set;
    Vue.prototype.$delete = del;

    // $watch
    Vue.prototype.$watch = function (expOrFn,cb,options) {};
  }
```