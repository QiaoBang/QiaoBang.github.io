---
layout:     post
title:      "JavaScript原型链的基本理解"
subtitle:   "一些思考和整理"
date:       2017-09-14
author:     "BigBangBro"
header-img: "img/post-bg-js-version.jpg"
tags:
    - JavaScript 
---


# 理解JavaScript原型和原型链

> “原型式集成”是JavaScript的的核心特征。————《JavaScript权威指南》

### 1. 原型规则

* 所有的引用类型都具有对象特性，即可自由扩展属性（null除外）

* 所有引用类型都有一个__proto__ //隐式原型属性，属性值是一个普通的对象
* 所有函数都有一个prototype//显示原型属性，属性值也是一个普通的对象

* 所有引用类型的__proto__属性值指向它的构造函数的prototype的属性值

* 当试图得到一个引用类型的某个属性时，如果这个对象本身没有这个属性，那么会去它的__proto__(即它构造函数的prototype)中寻找

### 实例
```
var obj={};obj.a=100;
var arr=[];arr.a=100;
function fn(){}
fn.a=100

console.log(obj.__proto__)
console.log(arr.__proto__)
console.log(fn.__proto__)

console.log(fn.prototype)

console.log(obj.__proto__===Object.prototype)   //true
```
#
```
//构造函数
function Foo(name,age){
this.name=name
}
Foo.prototype.alertName=function(){  //prototype是一个普通对象，可以扩展属性
alert(this.name)   
}    
//创建实例
var f=new Foo('zhangsan')
f.printName=function(){
console.log(this.name)
}
//测试
f.printName()    //zhangsan
f.alertName()    //f没有alertName属性,于是去f._proto_即Foo.prototype中查找

```
### *由对象调用原型中的方法，this指向对象*
```
//循环对象自身的属性
var item
for(item in f){
//高级浏览器已在for in中屏蔽了来自原型的属性
//但是这里建议还是加上这个判断以保证程序的健壮性
if(f.hasOwnProperty(item)){
    console.log(item)
}
}
```
## 2. 原型链
```
//在刚刚的代码中加入
f.toString()    //要去f.__proto__.__proto__中查找
```
#### 所有的引用类型都有```__proto__```属性，且```__proto__```属性值指向它的构造函数的prototype的属性值，所以当f不存在```toString```时，便会在```f.__proto__```即```Foo.prototype```中查询，而```Foo.prototype```中也没有找到```toString```。由于Foo.prototype也是一个对象，所以它隐式原型```__proto__```的属性值便指向它的构造函数Object的```prototype```的属性值。
```
//试一试
console.log(Object.prototype) 
console.log(Object.prototype.__proto__)    //为了避免死循环，所以此处输出null
```
![Alt text](https://github.com/QiaoBang/QiaoBang.github.io/blob/master/img/prototype.png?raw=true)

### instanceof

用于判断引用类型属于哪个构造函数的方法
```f instanceof Foo```的判断逻辑是f的```__proto__```一层层向上能否对应到```Foo.prototype```，再试着判断```f instanceof Object```

## 思考
### 1.如何准确判断一个变量是数组类型？
#### 使用instanceof
``` 
var arr=[]
arr instanceof Array    //true
typeof arr    //object    typeof无法准确判断是否是数组
```
### 2.写一个原型链继承的例子
```
//简单示例
//动物
function Animal(){
    this.eat=function(){
        console.log('Animal eat')
    }
}
//狗
function Dog(){
    this.bark=function(){
        console.log('Dog bark')
    }
}
Dog.prototype=new Animal()
//哈士奇
var hashiqi=new Dog()
```
### 
```
//封装一个DOM查询
function Elem(id){
    this.elem=document.getElementById(id)
}
Elem.prototype.html=function(val){
    var elem=this.elem
    if(val){
        elem.innerHTML=val
        return this    //链式操作
    }else{
        return elem.innerHTML
    }
}

Elem.prototype.on=function(type,fn){
  var elem=this.elem
  elem.addEventListener(type,fn)
  return this   //链式操作
}

var div1=new Elem('div1')
// console.log(div1.html());
// div1.html('<p>Hello</p>');
// div1.on('click',function(){
//   alert('clicked')
// });
div1.on('click',function(){alert('clicked')}).html('<p>链式操作</p>')
//在之前的函数中增加了return this，由div1调用时便会返回当前对象，即div1，便可以实现链式操作
```
### 3.描述new一个对象的过程
```
function Foo(name,age){
 // this={}
    this.name=name
    this.age=age
    this.class='class-1'
  //return this
}
var f=new Foo('zhangsan',20)
//var f1=new Foo('lisi',22) //创建多个对象
```
#### 创建一个新对象
#### this指向这个新对象``` this={}```
#### 执行代码，即对this赋值 ```this.xxx=xxx```
#### 返回this   ``` return this```

## 补充：
* 构造函数
```
function Foo(name,age){
  this.name=name
  this.age=age
  this.class='class-1'
//return this  //默认有这一行
}
var f=new Foo('zhangsan',20)
//var f1=new Foo('lisi',22) //创建多个对象

```
* 构造函数-扩展
```
var a={} //其实是var a=new Object()的语法糖
var b=[] //其实是var b=new Array()的语法糖
function Foo(){...} //其实是var Foo=new Function(...)
//使用instanceof判断一个函数是否是一个变量的构造函数
```
所有的 **引用类型（对象、数组、函数）** 都有构造函数，a的构造函数是Object()，b的构造函数是Array()，Foo的构造函数是Function()。所以假如想要判断一个变量是否为数组就可以使用
```
var a={}
a instanceof Array   //false
```
## 总结
>函数的原型对象constructor默认指向函数本身，原型对象除了有原型属性外，为了实现继承，还有一个原型链指针proto，该指针指向上一层的原型对象，而上一层的原型对象的结构依然类似，这样利用proto一直指向Object的原型对象上，而Object的原型对象用Object.prototype.proto = null表示原型链的最顶端，如此变形成了javascript的原型链继承，同时也解释了为什么所有的javascript对象都具有Object的基本方法。