1. 1. 1. ## 一、什么是作用域

         > 规定变量存放在哪些位置，以及怎么找到这些变量的规则。是配合引擎和编译器进行工作的变量存储地

         - 编译阶段，当编译器语句的解析时，声明某个变量，会先去作用域中查找是否有该值，否则在作用域中添加。

         - 执行阶段，引擎执行编译器生成的代码，使用LHS（赋值号左边）和RHS（右侧）的方式去查找作用域中对应的变量

         **为和区分LHS RHS**
         - 非strict模式下，LHS如果在所有作用域中找不到该变量，全局作用域会默认在全局中声明一个变量。
         - RHS则会报出ReferenceError

         ## 二、词法作用域

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
         ## 三、函数与块作用域

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
         > 在全局、局部代码**执行前**对当前作用域数据的预编译，生成VO和AO对象，供当前域下的执行语句的变量获取。

         1. 创建AO/VO对象
         2. 将形参和声明的变量添加到上诉对象
         3. 实参赋值到形参
         4. 函数声明添加到对象
         5. 添加`Scope`对应关系
         6. 添加`this`属性

         ## 五、 模块

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

         #### 现代的模块

         挖坑待填。。。

         ## 六、this

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
         > bind就是返回一个硬绑定apply的函数封装
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

         ## 七、构造函数

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

         ## 八、原型
         1. 提取共有属性

         2. `prototype`中的`constructor`指向当前函数，为了让继承的对象能找到自身的构造器

         3. `__proto__`指向`prototype`也能被修改

         4. new会进行隐式赋值  `__proto__ = prototype`

         5. 大多数对象的原型继承自`Object.prototype`，但由`Object.create为null`
         ```
         var a = Object.create(b) //a的原型为b   b可为null
         
         ```
         6. 原型重写：在对象访问通过原型链查找，优先使用较近的原型中的方法

         ## 九、继承
         缺点

         **1. 传统原型**

         * 过多继承了没用的属性
         ```
         Target.prototype = new Origin();
         ```

         **2. 构造函数**

         * 不能继承构造函数的原型

         * 每次继承都要调用父类方法
         ```
         function Target(){
            Target.prototype = Origin.call(this);
         }
         ```


         **3. 共享模式**

         * 改变子类原型时，父类也会随之改变

         ```
         Target.prototype = Origin.prototype
         ```

         **4. 圣杯模式**

         * 结合了前几种的优点

            ```
            var inherit = function(Target, Origin){
                var Middle = function(){}
                return function(){
                    Middle.prototype = Origin.prototype;
                    Target.prototype = new Middle(); //new 
                    Target.prototype.constructor = Target; //son的实例中constructor根据其构造函数原型的constructor而确定
                    Target.prototype.uber = Origin.prototype; //保存真正继承自哪个父类
                }
            }
            function Son(){}
            
            function Father(){}
              
            Father.prototype.hello = 'hello';
            
            inherit(Son, Father);
            
            var son = new Son();
            var father = new Father();
            console.log(son.hello); //'hello'
            console.log(father.hello); //'hello'
            ```

         ## 十、命名空间
         * 传统通过对象进行保存

         ```
         var a = {
             b : 1
         }
         
         var a1 = {
             b : 1
         }
         ```


         * 现在可以使用闭包进行编码
         ```
         var a = (function(){
             var b;
             return function(){
                 console.log(b)
             }
         }())
         ```

         ## 十一、对象枚举

         1. for in

         > 会遍历对象的__proto__，使用hasOwnProperty进行过滤。手动设置的对象会被遍历出，系统自带的不会遍历出

         2. in

         > 判断属性能不能在对象上访问到，包括原型

         3. instanceof 

         > 对象的原型链上有没有构造函数的prototype

         ## 十二、类数组
         ```
         var obj = {"0":1, "1":2, length:2, push: Array.prototype.push} 
         
         Array.prototype.push = function(target){
             this[this.length] = target;
             this.length++
         }
         ```
         1. 属性为索引
         2. length
         3. 有push, splice等数组操作属性


         ## 十三、异步

         1. `domTree`: 将文档中的节点一层层按深度优先解析成树形结构

         2. `cssTree`：`domTree`解析完成后等待异步加载的`cssTree`解析成一个树形结构

         3. `renderTree`：根据`cssTree`每个节点对对应的`domTree`的描述，形成一个`renderTree`，最后通过浏览器的渲染引擎绘制

         *注意*

         - dom节点的删除，增加
         - 宽高位置变化，`display:none -> block，offsetWidth offsetLeft`
           ...

         > 上述`domTree`改变会导致`renderTree` reflow重排，则又会进行上述1-3步骤，消耗性能

         - 颜色更变，背景图片...

         > 会使`renderTree` 进行`repaint`重绘，但性能相比重排消耗较少

         JS文件加载是阻塞的，与当前操作页面元素无关的JS文件可以进行异步加载

         1. `defer`: 异步执行，等到`domTree`解析完执行外部脚本或行内脚本 (ie8以下)
         2. `async`: 异步执行，只能加载外部执行脚本，加载完就执行
         3. 动态插入脚本：当创建`script`标签并且设置`src`后就已经异步下载（*注），插入到节点树后就直接执行（等待文件下载完毕后）

         *注

         *灯塔模式，创建`img`标签设置`src`进行资源的预加载，稍后取数据则直接在缓存中取*

         **加载时间线**

         * 键入URL后浏览器开启一个下载线程，然后通过http请求获取一个页面

         *在创建document对象后，围绕其readyState属性的三个状态`loading, interactice, complete`之间的处理过程。*

         * 创建`document`对象，UI线程开始解析以树的形式解析HTML元素，此时`document.readyState = 'loading'`
         * 解析`<img>`或者`<link>`的CSS标签浏览器都会创建线程异步下载，继续执行解析文档
         * 解析到`<script>`标签，UI线程挂起，JS引擎单线程执行，进行同步阻塞；若为异步属性再异步下载暂不执行（在异步script文件中无法使用`document.write()`）
         * 文档解析完后`document.readyState = 'interactive'`
         * `defer`属性的JS执行
         * `document`触发`DOMContentLoaded`事件，由脚本执行阶段转换为事件驱动阶段
         * 待img静态资源文件，`async`文件加载后，`document.readyState = 'complete'`
         * 处理用户输入等事件

         

         

         

         ----

         客户端Javascript
         ## DOM

         ```
         - document --> HTMLDocument.prototype --> Document.prototype
         - <html> --> HTMLHtmlElement.prototype --> Element.prototype
         - 终端为Object.prototype
         ```


         ```
         Document.prototype.abc = 'abc'
         "abc"
         document.abc
         "abc"
         HTMLDocument.prototype.abc = 'a'
         "a"
         document.abc
         "a"
         
         delete HTMLDocument.prototype.abc
         true
         document.abc
         "abc"
         ```

         增
         - createElement
         - createTextNode
         - createComment
         - createDocumentFragment

         删
         - child.remove()
         - parents.removeChild(element) //剪切

         插
         - insertBefore
         - appendChild() //剪切

         替换
         - node.replaceChild(new, old)

         # Window
         **滚动条距离**

         ie9上

         > window.pageXOffset

         ie9下

         > document.body.scrollTop + document.documentElement.scrollTop //两者只有一个有值，另一个为0

         **视口尺寸**

         ie8上

         > window.innerHeight

         ie8下  //高版本两者都有值

         - 标准模式

         > document.documentElement.clientHeight  //与innerHeight相同

          - 怪异/混杂模式

         > document.body.clientHeight

         **偏移距离**

         `element.offsetTop  //相对最近的父类绝对定位的值`

         # 事件

         **三种方式**

         ```
         div.onclick = function(){}  //句柄
         
         div.addEventListener(event, function, type)
         
         div.attacheEvent()
         ```

         *注意*

         1. 先捕获
         2. 执行事件程序 (和绑定执行顺序有关)
         3. 冒泡

         *阻止默认事件*

         1. 只有句柄才能使用`return false`阻止默认事件
         2. `event.preventDefault()、event.returnValue = false`