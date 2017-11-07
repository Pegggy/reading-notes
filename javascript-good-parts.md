# Function
## 调用
调用一个函数会暂停当前函数的执行，把传递控制权和参数给新函数。除了声明时定义的形参，每个函数还会接受两个附加参数：this 和 arguments。参数 this 的值取决于调用模式。在 JavaScript 中有四种调用模式：方法调用模式、函数调用模式、构造器调用模式和 apply 调用模式。这些模式在初始化 this 上存在差异。

### 方法调用模式
当函数被保存为对象的一个属性时，我们称之为一个方法。当一个方法被调用时，this 被绑定到该对象。
```js
function f1(){
  console.log(this)
}
var obj = {
	a : 1,
	fn: f1
}

obj.fn();
// {a: 1, fn: ƒ}
```
方法调用可以使用 this 访问自己所属的对象，所以他能从对象中取值或对值进行修改。this 对象的绑定发生在调用的时候，通过 this 可以取得它们所属对象的上下文的方法称为公共方法。

```js
function f2(){
	this.a += 1
	console.log(this.a)
}

obj.f2 = f2
obj.f2();
// {a: 2, fn: ƒ, f2: ƒ}
```
### 函数调用模式
当函数并非一个对象的属性时，它就是被当做一个函数来调用的。此时 this 被绑定到全局对象上。
```js
var a = 5
obj.add2 = function(){
	var helper = function(){
		this.a = add(this.a,this.a)
    }
	helper()
}
obj.add2()
console.log(a)
// 10
```
当函数作为一个函数来调用时，this 指向的是 window,此时改变的是 window.a 的值。
