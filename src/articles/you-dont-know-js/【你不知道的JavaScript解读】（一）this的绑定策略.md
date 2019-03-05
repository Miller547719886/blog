## 四种绑定方式

在本书中的第二部分（this和对象原型）介绍了this的四种绑定方式，按照优先级先后顺序如下所示：

1. new绑定
2. 显式绑定
3. 隐式绑定
4. 默认绑定

而咱们在实际使用场景中用到的方法还不只这些，会存在一些变种的使用方式，咱们往下看。

## 隐式绑定

首先咱们看看下面代码以回顾下隐式绑定的概念：

```javascript
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

可以看到，所谓`隐式`的概念就是说： 当一个函数**在调用时**被一个对象所“拥有”或“包含”，那么这个函数的this将指向“拥有”或“包含”它的对象，并且只有对象属性引用链的最后一层会有这个效果。

### 隐式绑定丢失

this绑定最常让人沮丧的事情之一，就是当一个 隐含绑定 丢失了它的绑定，这通常意味着它会退回到 默认绑定（绑定到全局对象）， 如果在严格模式下，结果是undefined。

```javascript
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2,
	foo: foo
};

var bar = obj.foo; // 函数引用！

var a = "oops, global"; // `a`也是一个全局对象的属性

bar(); // "oops, global"
```

这种现象同样适用于函数传参，此时我们理解函数传参会进行一次隐式的赋值，无论接收回调的是内置函数还是你自己写的函数：

```javascript
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2,
	foo: foo
};

var a = "oops, global"; // `a`也是一个全局对象的属性

function doFoo(fn) {
	// `fn` 只不过`foo`的另一个引用

	fn(); // <-- 调用点!
}

doFoo( obj.foo ); // "oops, global"

setTimeout( obj.foo, 100 ); // "oops, global"
```

## 显式绑定

### call，apply

为了摆脱隐式绑定丢失的困扰，显式绑定应运而生，内置函数`call`与`apply`就是显示绑定的典型体现，就像下面这样：

```javascript
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2
};

foo.call( obj ); // 2
```

call与apply的唯一区别就在于其传递给待执行函数的参数的形式是多个参数还是数组。

书中还特意提到了如果你传递一个简单原始类型值（string，boolean，或 number类型）作为this绑定，那么这个原始类型值会被包装在它的对象类型中（分别是new String(..)，new Boolean(..)，或new Number(..)）。这通常称为“装箱”。

### api传递上下文

除了call和apply两个内置方法外，一部分内置函数也支持传入对象作为其回调函数的上下文：

```javascript
var obj = {
  a: 2
}
var a = 1
var l = [1,2,3]
l = l.map(function(item) {
  item += this.a
  return item
}, obj)

// [3, 4, 5]
```

像数组中的forEach,map,filter,some,every都支持传入`上下文对象`来改变回调函数种的this。各位FEer们在撸代码的时候如果遇到了内置函数需要传入`回调函数`作为参数，不妨在MDN上查一下是否支持传入`上下文对象`，说不定能节省你不少时间，哈哈。

## 硬绑定

call与apply的限制在于，他们将直接执行回调函数，而我们有时需要得到一个函数，其内部this被强制绑定在某个对象上，且绝对不会被改变。为了满足这个需求，咱们在ES5标准里发现了bind函数，很好的做到了这点：

```javascript
function foo(something) {
	console.log( this.a, something );
	return this.a + something;
}

// 简单的`bind`帮助函数
function bind(fn, obj) {
	return function() {
		return fn.apply( obj, arguments );
	};
}

var obj = {
	a: 2
};

var bar = bind( foo, obj );

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

经常会有笔试题要求手写实现一个bind函数，那么上面就是一个最初级的例子，不过咱们还是要看看MDN上的比较完善的实现：

```javascript
if (!Function.prototype.bind) { // 检查当前环境是否内置bind函数
  Function.prototype.bind = function(oThis) { // 没有则在函数原型上定义一个bind函数，接收一个对象作为上下文
    if (typeof this !== 'function') { // 如果调用者不为函数则抛出错误
      throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable');
    }

    var aArgs   = Array.prototype.slice.call(arguments, 1), // 获取bind函数的"预置参数"
        fToBind = this, // 获得函数原始的上下文
        fNOP    = function() {}, // 声明一个空函数以继承其原型链
        fBound  = function() { // 声明要输出的被“合理”处理了上下文的函数
          return fToBind.apply(this instanceof fBound
                 ? this // 返回的fBound被当做new的构造函数调用
                 : oThis, // 返回的fBound被当做普通函数调用
                 aArgs.concat(Array.prototype.slice.call(arguments))); // 合并预置参数与fBound传入的参数
        };

    if (this.prototype) {
      // Function.prototype doesn't have a prototype property
      fNOP.prototype = this.prototype;
    }
    // 下行的代码使fBound.prototype是fNOP的实例,因此
    // 返回的fBound若作为new的构造函数,new生成的新对象作为this传入fBound,新对象的__proto__就是fNOP的实例
    fBound.prototype = new fNOP();

    return fBound;
  };
}
```

关于上面bind实现对于new绑定函数的prototype的处理，咱们在后面关于原型的文章种还会做进一步讨论。

## 软绑定

咱们通过下面例子看看硬绑定会有哪些限制：

```javascript
function foo() {
	console.log( this.a );
}


var obj = {
	a: 1
};
var obj2 = {
	a: 2
};
var obj3 = {
	a: 3
};

var bar = foo.bind(obj);
var fooInstance = new (foo.bind(obj))()

bar(); // 1
bar.call(obj2) // 1
obj3.bar = bar
obj3.bar() // 1
fooInstance.a // undefined
```

上面的示例说明，硬绑定也太“硬”了，只要被bind绑定过一次，其内部this将终身不变（除了new绑定以外），也就是不够灵活，被bind处理过的函数将无法使用隐式绑定和显式绑定改变其内部this指向。有没有更灵活的办法？有的，看下面的`softBind`实现:

```javascript
if (!Function.prototype.softBind) {
  Function.prototype.softBind = function(obj) {
    var fn = this,
      curried = [].slice.call( arguments, 1 ),
      bound = function bound() {
        return fn.apply(
          (!this ||
            (typeof window !== "undefined" &&
              this === window) ||
            (typeof global !== "undefined" &&
              this === global)
          ) ? obj : this,
          curried.concat.apply( curried, arguments )
        );
      };
    bound.prototype = Object.create( fn.prototype );
    return bound;
  };
}
```

仔细阅读上面函数的实现逻辑能发现，它与硬绑定的区别在于它判断this是否为undefined或者全局变量，是则变更为传入的函数，否则不改变this指向，利用这个逻辑，显式绑定与隐式绑定就能发挥作用啦。

## 总结

this的绑定方式分为`new绑定`,`显式绑定`,`隐式绑定`,`默认绑定`四种，按先后顺序排列优先级，其中隐式绑定存在触发条件苛刻的问题，于是出现了显式绑定（`call`与`apply`），而显式绑定无法处理绑定丢失的问题，又出现了硬绑定的实现（`bind`），而硬绑定在某些场景下不够灵活，所以又有了软绑定（`softBind`）。我们需要充分理解这几种绑定的使用场景并合理的利用他们。
