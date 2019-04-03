### 一、什么是作用域
> 规定变量存放在哪些位置，以及怎么找到这些变量的规则。是配合引擎和编译器进行工作的变量存储地

- 编译阶段，当编译器语句的解析时，声明某个变量，会先去作用域中查找是否有该值，否则在作用域中添加。

- 执行阶段，引擎执行编译器生成的代码，使用LHS（赋值号左边）和RHS（右侧）的方式去查找作用域中对应的变量

**为和区分LHS RHS**
- 非strict模式下，LHS如果在所有作用域中找不到该变量，全局作用域会默认在全局中声明一个变量。
- RHS则会报出ReferenceError

### 二、词法作用域
> 在编译器分析函数被声明的位置所定义的作用域

- eval 会改变词法作用域
- with 会将对象作为语句块的临时作用域，肯能造成全局作用域泄露
```
function foo(obj) {
	with (obj) {
		a = 2;
	}
}

var o1 = {
	a: 3
};

var o2 = {
	b: 3
};

foo( o1 );
console.log( o1.a ); // 2

foo( o2 );
console.log( o2.a ); // undefined
console.log( a ); // 2 -- 哦，全局作用域被泄漏了！
```
### 三、函数与块作用域
**最低权限原则**
> 一些函数的私有方法需在对应的函数上下文中进行声明，可避免其它作用域也能访问到该方法的声明

```
function doSomething(a) {
	function doSomethingElse(a) {
		return a - 1;
	}

	var b;

	b = a + doSomethingElse( a * 2 );

	console.log( b * 3 );
}

doSomething( 2 ); // 15
```

**函数声明污染作用域**
> 当不想让执行的函数污染外层作用域，可以使用函数表达式的方式解决 `(function name(){})()` 或者 `(function name(){}())`

```
var a = 2;

(function foo(){ // <-- 插入这个

	var a = 3;
	console.log( a ); // 3

})(); // <-- 和这个

console.log( a ); // 2
```

**函数的匿名与命名**

优点：
- 书写简单快捷

缺点：
- 当要递归该匿名函数时，只用使用arguments.callee（已废弃）
- 栈轨迹上匿名函数没有直白的名称显示，加大调试难度

**同一模块定义 UMD**
> 将功能性函数以参数的形式传递到另一个参数中被调用

```
var a = 2;

(function IIFE( def ){
	def( window );
})(function def( global ){

	var a = 3;
	console.log( a ); // 3
	console.log( global.a ); // 2

});
```

**模拟块作用域**
- with
- try catch

**执行上下文**
> 保存当前全局或者局部代码执行前数据的对象 context

该对象有三个重要属性
1. 变量对象 函数 arguments context.vo ao
2. 作用域链 context.scope
3. this context.this

## 四、作用域闭包
> 一个能记住它词法作用域，且在词法作用域外被调用的函数
```
function a(d){
    function b(){
        console.log(d)
    }
    return b
}

a(1)()
```

**经典例子**

*`for i`时，只在该作用域顶部声明一个`i`，进行迭代的循环累加*
```
for (var i=1; i<=5; i++) {
	setTimeout( function timer(){
		console.log( i );
	}, i*1000 );
}
```

```
for (var i=1; i<=5; i++) {
	(function (i){
    	setTimeout( function timer(){
    		console.log( i );
    	}, i*1000 );	    
	})(i)
}
```

*`for let`时，会在每次作用域块中都声明一个`let`，且由上一次`let`进行初始化赋值*
```

for (let i=1; i<=5; i++) {
	setTimeout( function timer(){
		console.log( i );
	}, i*1000 );	    
}

for (var i=1; i<=5; i++) {
	{
	    let j = i;  //等同于
    	setTimeout( function timer(){
    		console.log( j );
    	}, i*1000 );	    
	}
}

```
**执行上下文**
> 在全局、局部代码执行前对当前作用域中数据的初始化

### 模块
> 用闭包的性质返回一个对象（API），这个对象包含了对内部数据的操作
```
function CoolModule() {
	var something = "cool";
	var another = [1, 2, 3];

	function doSomething() {
		console.log( something );
	}

	function doAnother() {
		console.log( another.join( " ! " ) );
	}

	return {
		doSomething: doSomething,
		doAnother: doAnother
	};
}

var foo = CoolModule();

foo.doSomething(); // cool
```
- 单例模式
> 模块仅生产一个实例供调用，在使用前默认调用一次
```
var modules = (function(){})()
```

### 现代的模块
挖坑待填。。。

## 五、this
> this不是编写时绑定，而是运行时绑定，完全根据调用点进行对象的绑定

**调用栈**
> 执行到当前位置执行的所有函数的堆栈，调用点就是在当前执行函数之前调用的

找到调用点后，根据四种规则顺序判断this的真正指向

1. 默认绑定
> 当函数没有被任何修饰的引用调用，则会默认绑定到window上（strict mode为undefined）

*注意，strict模式是基于函数运行时的作用域而不是调用点的作用域*

```
function foo() {
	console.log( this.a );
}

var a = 2;

(function(){
	"use strict";

	foo(); // 2
})();
```

2. 隐式绑定
> 以对象方法的直接调用，该方法绑定的this为此对象

*只有对象属性引用链的最后一层是影响调用点的*

```
function foo() {
	console.log( this.a );
}

var obj2 = {
	a: 42,
	foo: foo
};

var obj1 = {
	a: 2,
	obj2: obj2
};

obj1.obj2.foo(); // 42
```

*隐藏丢失，当声明的变量指向了对象的方法，该变量调用时相当于默认绑定*

```
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2,
	foo: foo
};

var bar = obj.foo; // 函数引用！

var a = "oops, global"; // `a` 也是一个全局对象的属性

bar(); // "oops, global"
```

同样的还有回调函数
```
function foo() {
	console.log( this.a );
}

function doFoo(fn) {
	// `fn` 只不过 `foo` 的另一个引用

	fn(); // <-- 调用点!
}

var obj = {
	a: 2,
	foo: foo
};

var a = "oops, global"; // `a` 也是一个全局对象的属性

doFoo( obj.foo ); // "oops, global"
```



函数也是对象，也可以进行属性的操作
```
function foo(num) {
	console.log( "foo: " + num );

	// 追踪 `foo` 被调用了多少次
	foo.count++;
}

foo.count = 0;
```

3. 显示绑定
> 使用 call，apply， bind
bind就是返回一个硬绑定apply的函数封装
```
function bind(fn, obj){
    return function(){
        return fn.apply(obj, arguments)
    }
}
```

4. new绑定
> 用new操作符进行对象的构造

new时会进行
- 创造一个空对象
- 对象会被接入原型链
- 将this与该对象绑定
- 如果函数没有返回对象则返回该对象

```
function foo(a) {
	this.a = a;
}

var bar = new foo( 2 );
console.log( bar.a ); // 2
```

**this赋值执行顺序优先级从上至下依次变大**

- 特例

1. 间接：当使用圆括号的方法赋值调用时，圆括号会将该句解析为函数表达式，会默认使用默认绑定

```
function foo() {
	console.log( this.a );
}

var a = 2;
var o = { a: 3, foo: foo };
var p = { a: 4 };

o.foo(); // 3
(p.foo = o.foo)(); // 2
```
2. 箭头函数，箭头函数的this不能被改变，默认指向的是词法环境上下文中的this
```
function foo() {
	setTimeout(() => {
		// 这里的 `this` 是词法上从 `foo()` 采用
		console.log( this.a );
	},100);
}

var obj = {
	a: 2
};

foo.call( obj ); // 2
```

chrome tool中的restart frame可以跳回指定的调用栈

## js构造函数
> 构造函数就是普通函数

**包装类**
> 当基础类型发生属性或者方法的访问，js引擎会默认进行隐式的包装类，随机销毁

```
var aa = '11';
console.log(aa.length);

aa.len = 1;
//new String(a).len = 3; delete
console.log(aa.len); //undefined

```

## 原型（prototype）
> 函数的祖先
**应用**
1. 提取共有属性

prototype中的constructor指向当前函数，为了让继承的对象能找到自身的构造器

__proto__指向prototype也能被修改

new会进行赋值  __proto__ = prototype

传递的是指针，各种修改不会影响

var a = Object.create(b)
> a的原型为b   b可为null
 
所有对象都有原型继承自Object.prototype，但由Object.create为null
 
**原型重写**
> 在对象访问通过原型链查找，优先使用较近的原型中的方法
