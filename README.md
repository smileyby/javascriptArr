JavaScript数组所有API全解密
=========================

数组是一种非常重要的数据类型，它语法简单、灵活、高效。在多数编程语言中，数组都充当着至关重要的角色，以至于很难想象没有数组的编程语言会是什么模样。特别是javascript，它天生灵活，有进一步发挥了数组的特长，丰富了数组的使用场景。可以好不夸张地说，不深入地了解数组，不足以写javascript。

截止ES7规范，数组共包含33个标砖的API方法和一个费标砖的API方法，使用场景和使用方案纷繁复杂，其中有不少浅坑，神坑甚至神坑。下面将从Array构造器及ES6新特性开始，逐步帮助你掌握数组。

声明：一下未特别声明的方法均为ES5以实现方法。

## Array构造器

Array构造器用于构建一个新的数组。通常，我们推荐使用字面量创建数组，这是一个好习惯，但是总有对象字面量乏力的时候，比如说，我想创建一个长度为8的空数组。请比较如下两种方式：

```js

// 使用Array构造器
var a = Array(8); //[undefined * 8]
// 使用对象字面量
var b = [];
b.length = 8; // [undefined * 8]

```

Array构造器明显要简洁一些，当然你也许会说，对象字面量也不错，那我保持沉默。

如上，我是用`Array(8)`而不是`new Array(8)`，这会影响吗？实际上，并没有影响，这得益于Array构造器内部对this指针的判断，[ELS5_HTML规范](http://ecma262-5.com/ELS5_HTML.htm#Section_15.4.1)是这么说的：

```

When Array is called as a function rather than as a constructor, it creates and initialises a new Array object. Thus the function call Array(…) is equivalent to the object creation expression new Array(…) with the same arguments.

```

从规范来看，浏览器内部大致做了如下类似的实现：

```js

function Array() {
	// 如果this不是Array的实例，那就重新new一个实例
	if(!(this instanceof arguments.callee)) {
		return new arguments.calee();
	}
}

```

关于Array构造器语法的介绍：

Array构造器根据参数长度不同，有如下两种不同的处理：

*	new Array(arg1, arg2...),参数长度为0或长度等于2时，传入参数将按照顺序依次称为新数组的地0至N项（参数长度为0时，返回空数组）
*	new Array(len)，当len不是数值时，处理同上，返回一个只包含len元素一项的数组；当len为数值时，根据如下规范，len最大不能超过32位无符号整型，即需要小于2的32次方（len最大为`Math.pow(2,32) -1 或 -1>>>0`），否则将抛出RangeError

```

If the argument len is a Number and ToUint32(len) is equal to len, then the length property of the newly constructed object is set to ToUint32(len). If the argument len is a Number and ToUint32(len) is not equal to len, a RangeError exception is thrown.

```

以上，请注意Array构造器对于单个数值参数的特殊处理，如果仅仅需要使用数组包裹若干参数，不放使用Array.of，具体请看下一节。

## ES6新增的构造函数方法

剑雨数组的常用性，ES6专门扩展了数组构造器`Array`，新增2个方法：`Array.of`、`Array.from`。

### Array.of

Array.of用于将参数一次转化为数组中的一项，然后返回这个新数组，而不管这个参数是数字还是其他。它基本上与Array构造器功能一致，唯一的区别就在单个数字参数的处理上。如下：

```js

Array.of(8.0); //[8]
Array(8.0); // [undefined * 8]

```

参数为多个，或单个参数不是数字时，Array.of与Array构造器等同

```js

Array.of(8.0, 5); // [8]
Array(8.0, 5); // [8, 5]
Array.of('8'); // ['8']
Array('8'); //['8']

```

因此，若是需要使用数组包裹元素，推荐优先使用Array.of方法

目前，以下版本的浏览器提供对Array.of的支持

<table>
	<tr>
		<td>Chrome</td>
		<td>Firefox</td>
		<td>Edge</td>
		<td>Safari</td>
	</tr>
	<tr>
		<td>45+</td>
		<td>25+</td>
		<td>✔️</td>
		<td>9.0+</td>
	</tr>
</table>

计时其他版本浏览器不支持也不必担心，由于Array.of与Array构造器的这种高度相似性，实现一个polyfill十分简单。如下：

```js

if (!Array.of){
	Array.of = function(){
		return Array.prototype.slice.call(arguments);
	}
}

```

### Array.from

语法：Array.from(arrayLike[, processingFn[, thisArg]])

Array.from的设计初衷是快速便捷的基于其他对象创建新数组，准确来说就是从一个类似数组的可迭代对象创建一个新的数组实例，通俗一点，只要一个对象有迭代器，Array.from就能把它变成一个数组（当然，是返回新的数组，不改变原对象）。

从语法上看，Array.form拥有三个形参，第一个为类似数组的对象，必选。第二个为加工函数，新生成的数组会经过该函数的加工再返回。第三个为this作用于，表示加工函数执行时this的值。后两个参数都是可选的。我们来看看用法。

```js

var obj = {0: 'a', 1: 'b', 2: 'c', length: 3};
Array.from(obj, function(value, index){
	console.log(value, index, this, arguments.length);
	return value.repeat(3); // 必须制定返回值，否则返回undefined
}, obj);

```

执行结果如下：

![](images/1.png)

可以看到加工函数的this作用域被obj对象取代，也可以看到加工函数默认拥有两个形参，分别为迭代当前元素的值和其索引。

注意，一旦使用加工函数，必须明确指定返回值，否则将饮食返回undefined，最终生成的数组编程一个只包含若干个undefined元素的空数组。

实际上，如果不需要指定this，加工函数完全可以是一个箭头函数，上述代码可以简化如下：

```js

Array.from(obj, (value) => value.repeat(3));

```

除了上述obj对象外，拥有迭代器的对象还包括这些：`String`、`Set`、`Map`、`arguments`等。

Array.from统统可以处理，如下所示：

```js

// String
Array.from('abc'); // ['a', 'b', 'c']
// Set
Array.from(new Set(['abc', 'def']));
// Map
Array.from(new Map([[1, 'abc'], [2, 'def']]));
// 提升讷航的类数组对象arguments
function fn(){
	return Array.from(arguments)
}
fn(1, 2, 3); //[1, 2, 3]

```

到这里你可能以为Array.from就讲完了，实际上还有一个重要的扩展场景必须提一下。比如说生成一个从0到指定数字的新数组，Array.from就可以轻易做到。

```js

Array.from({lentgh: 10}, (v, i) => i); // [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

```

后面我们将会看到，利用数组的keys方法实现上述功能，可能还要简单一些。

目前，一下版本浏览器提供了对Array.from的支持

<table>
	<tr>
		<td>Chrome</td>
		<td>Firefox</td>
		<td>Edge</td>
		<td>Opera</td>
		<td>Safari</td>
	</tr>
	<tr>
		<td>45+</td>
		<td>32+</td>
		<td>✔️</td>
		<td>✔️</td>
		<td>9.0+</td>
	</tr>
</table>

## Array.isArray

顾名思义，Array.isArray用来判断一个变量是否是数组类型。JS的弱类型机制导致判断变量类型是初级前端开发者的必考题，一般我都会将其作为考察候选人第一题，然后基于此展开。在ES5提供该方法之前，我们至少有如下2种方式去判断一个值是否是数组：

```js

var a = [];
// 1.基于instanceof
a instanceof Array
// 2.基于constructor
a.constructor === Array
// 3.基于Object.prototype.isPrototypeOf
Array.prototype.idPrototypeOf(a);
// 4.基于getPrototypeOf
Object.getPrototypeOf(a) === Array.prototype;
// 5.基于Object.prototype.toString
Object.prototype.toString.apply(a) === '[object Array]'

```

以上，除了`Object.prototype.toString`外，其他方法都不能正确判断变量的类型。

要知道，代码的运行环境十分复杂，一个变量可能使用浑身解数去迷惑它的创造者，且看：

```js

var a = {
	_proto_: Array.prototype
};
// 分别在控制台试运行以下代码
// 1.基于instanceof
a instanceof Array; // true
// 2.基于constructor
a.constructor === Array; // true
// 3.基于Object.prototype.isPrototypeOf
Array.prototype.isPrototypeOf(a); //true
// 4.基于getPrototypeOf
Object.getPrototypeOf(a) === Array.prototype; // true
 
```

以上，4种方法将会全部返回`true`，为什么呢？我们只是手动指定了某个对象的`_proto_`属性为`Array.prototype`，便导致了该对象继承了Array对象，这种好不负责的继承方式，使得基于继承的判断方案瞬间土崩瓦解。

不仅如此，我们还知道，Array是堆数据，变量指向的只是它的引用地址，因此每个页面的Array对象引用的地址都是一样的。iframe中声明的数组，它的构造函数是iframe中的Array对象。如果在iframe声明了一个数组`X`，将其赋值给父页面的变量`Y`，那么在父页面使用`y instanceof Array`，结果一定是`false`的。而最后一种返回的是字符串，不会存在引用问题。实际上多页面或系统之间的交互只有字符串能够畅行无阻。

相反，使用Array.isArray则非常简单，如下：

```js

Array.isArray([]); // true
Array.isArray({0: 'a', length: 1}); // false

```

目前，一下版本浏览器提供对Array.isArray的支持

<table>
	<tr>
		<td>Chrome</td>
		<td>Firefox</td>
		<td>IE</td>
		<td>Opera</td>
		<td>Safari</td>
	</tr>
	<tr>
		<td>5+</td>
		<td>4+</td>
		<td>9+</td>
		<td>10.5+</td>
		<td>5+</td>
	</tr>
</table>

实际上，通过`Object.prototype.toString`去判断一个值的类型，也是个大主流库的标准。因此Array.isArray的polyfill通常长这样：

```js

if(!Array.isArray){
	Array.isArray = function(arg) {
		return Object.prototype.toString.call(arg) === '[object Array]';
	}
}

```

## 数组推导

ES6对数组的增强不止是体现在api上，还包括语法糖。比如说`for of`，它就是借鉴其他语言而成的语法糖，这种基于原数组使用`for of`生成新数组的语法糖，叫做数组推导。数组推导最初起草在ES6的草案中，但在第27版（2014年8月）中被移除，目前只有firefox v30+支持，推导有风向，使用需谨慎。索性，如今这些语言都还支持推导：CoffeeScript、Python、Haskell、Clojure，我们可以从中一窥端倪。这里我们以python的`for in`推导打个比方：

```

# python for in 推导
a = [1, 2, 3, 4]
print [i * i for i in a if i == 3] # [9]

```

如下是SpiderMonkey引擎（Firefox）之前基于ES4规范实现的数组推导(与python的推导十分相似)：

```
	
[i * i for (i of a)] // [1, 4, 9, 16]

```

ES6中数组有关的for of在ES4的基础上进一步演化，for关键字居首，in在中间，最后才是运算表达式。如下：

```

[for (i of [1, 2, 3, 4]) i * i] // [1, 4, 9, 16]

```

同python的示例，ES6中数组有关的for of也可以使用if语句：

```

// 单个if
[for (i of [1, 2, 3, 4]) if (i == 3) i * i] // [9]
// 甚至是多个if
[for (i of [1, 2, 3, 4]) if (i > 2) if (i < 4) i * i] // [9]

```

更为强大的是，ES6数组推导还允许多重for of。

```

[for (i of [1, 2, 3]) for (j of [10, 100]) i * j] // [10, 100, 20, 200, 30, 300]

```

甚至，数组推导还能够嵌入另一个数组推导中。

```
	
[for (i of [1, 2, 3]) [for (j of [10, 100]) i * j] ] // [[10, 100], [20, 200], [30, 300]]


```

对于上述两个表达式，前者和后者唯一的区别，就在于后者的第二个推导是先返回数组，然后与外部的推导再进行一次运算。

除了多个数组推导嵌套外，ES6的数组推导还会为每次迭代分配一个新的作用域（目前Firefox也没有为每次迭代创建新的作用域）：

```

// ES6规范
[for (x of [0, 1, 2]) () => x][0]() // 0
// Firefox运行
[for (x of [0, 1, 2]) () => x][0]() // 2

```

通过上面的实例，我们看到使用数组推导来创建新数组比forEach，map，filter等遍历方法更加简洁，只是非常可惜，它不是标准规范。

ES6不仅新增了对Array构造器相关API，还新增了8个原型的方法。接下来我会在原型方法的介绍中穿插着ES6相关方法的讲解，请耐心往下读。

## 原型

继承的常识告诉我们，JS中素有的数组方法均来自于Array.prototype，和其他构造函数一样，你可以通过扩展`Array`的`prototype`属性上的方法来给所有数组实例增加方法。

值得一说的是，Array.prototype本身就是一个数组

```js

Array.isArray(Array.prototype) // true
console.log(Array.prototype.length); //0

```

以下方法可以进一步验证：

```js

console.log([]._proto_.length); // 0
console.log([]._proto_); // [Symbol(Symbol.unscopables): Object]

```

有关Symbol(Symbol.unscopables)的知识，这个理不在详述，具体请移步后续章节。

## 方法

数组远行提供的方法非常之多，主要分为三种，一种是会改变自身值的，一种是不会改变自身值的，另外一种就是遍历方法。

由于Array.prototype的某些属性被设置为[[DontEnum]]，天赐不能用一般的方法进行遍历，我们可以通过如下方法获取Array.prototype的所有方法：

```js

Object.getOwnPropertyNames(Array.prototype); // ["length", "constructor", "toString", "toLocaleString", "join", "pop", "push", "reverse", "shift", "unshift", "slice", "splice", "sort", "filter", "forEach", "some", "every", "map", "indexOf", "lastIndexOf", "reduce", "reduceRight", "copyWithin", "find", "findIndex", "fill", "includes", "entries", "keys", "concat"]

```

### 改变自身值的方法（9个）

基于ES6，改变自身值的方法一共有9个，分别是pop、push、reverse、shift、sort、splice、unshift，以及两个ES6新增的方法copyWithin和fill。

对于能改变自身值的数组方法，日常开发中需要特别注意，尽量便在玄幻便利中去改变原数组的项。接下来我们一起来深入了解这些方法。

#### pop

pop()方法删除一个数组中的左后一个元素，并且返回这个元素。如果是栈的话，这个过程就是栈顶弹出。

```js

var array = ["cat", "dog", "cow", "chicken", "mouse"];
var item = array.pop();
console.log(array); // ["cat", "dog", "cow", "chicken"]
console.log(item); // mouse

```

由于设计上的巧妙，pop方法可以应用在类数组对象上，如下：

```js

var o = {0:"cat", 1:"dog", 2:"cow", 3:"chicken", 4:"mouse", length:5}
var item = Array.prototype.pop.call(o);
console.log(o); // Object {0: "cat", 1: "dog", 2: "cow", 3: "chicken", length: 4}
console.log(item); // mouse

```

但如果类数组对象不具有length属性，那么该对象被创建length属性，length属性值为0.如下：

```js

var o = {0:"cat", 1:"dog", 2:"cow", 3:"chicken", 4:"mouse"}
var item = Array.prototype.pop.call(o);
console.log(array); // Object {0: "cat", 1: "dog", 2: "cow", 3: "chicken", 4: "mouse", length: 0}
console.log(item); // undefined

```

#### push

push()方法添加一个或者多个元素到数组末尾，并返回数组新的长度。如果是栈的话，这个过程就是栈顶压入。

语法：arr.push(element1,...,elementN)

```js

var array = ["football", "basketball", "volleyball", "Table tennis", "badminton"];
var i = array.push("golfball");
console.log(array); // ["football", "basketball", "volleyball", "Table tennis", "badminton", "golfball"]
console.log(i); // 6

```

同pop方法一样，push方法也可以应用到类数组对象上，如果length不能被转成一个数值或者不存在length属性，则插入的元素索引为0，且length属性不存在时，将会创建它。

```js

var o = {0: "football", 1: "basketball"};
var i = Array.prototype.push.call(o, "golfball");
console.log(o); // Object {0: "golfball", 1: "basketball", length: 1}
console.log(i); // 1

```

实际上，push方法是根据length属性来决定从哪里开始插入给定值。

```js

var o = {0:"football", 1:"basketball",length:1};
var i = Array.prototype.push.call(o,"golfball");
console.log(o); // Object {0: "football", 1: "golfball", length: 2}
console.log(i); // 2

```

利用push根据length属性插入元素的特点，可以实现数组的合并，如下：

```js

var array = ["football", "basketball"];
var array2 = ["volleyball", "golfball"];
var i = Array.prototype.push.apply(array,array2);
console.log(array); // ["football", "basketball", "volleyball", "golfball"]
console.log(i); // 4

```

#### reverse

reverse()方法颠倒数组中元素的位置，第一个会成为最后一个，最后一个会成为第一个，该方法返回对数组的引用。

语法：arr.reverse()

```js

var array = [1,2,3,4,5];
var array2 = array.reverse();
console.log(array); // [5,4,3,2,1]
console.log(array2 === array) // true

```

同上，reverse也是鸭式辩型的受益者，调到元素的范围受length属性制约。如下：

```js

var o = {0:"a", 1:"b", 2:"c", length:2};
var o2 = Array.prototype.reverse.call(o);
console.log(o); // Object {0: "b", 1: "a", 2: "c", length: 2}
console.log(o === o2); // true

```	

如果length属性小于2或者length属性不为数值，那么原类数组对象将没有变化。及时length属性不存在，该对象也不会去创建length属性。特别的是，当length属性较大时，类数组对象的【索引】会尽可能的项length看齐。如下：

```js

var o = {0:"a", 1:"b", 2:"c",length:100};
var o2 = Array.prototype.reverse.call(o);
console.log(o); // Object {97: "c", 98: "b", 99: "a", length: 100}
console.log(o === o2); // true

```	

#### shift

shift()方法删除数组第一个元素，并返回这个元素。如果是栈的话，这个过程就是栈底弹出。

语法：arr.shift()

```js

var array = [1,2,3,4,5]
var item = array.shift();
console.log(array); // [2,3,4,5]
console.log(item); // 1

```								
同样受益于鸭式辩型，对于类数组对象，shift仍然能够处理。如下：

```js

var o = {0:"a",1:"b",2:"c",length:3};
var item = Array.prototype.shift.call(o);
console.log(o); // Object {0:"b",1:"c",length: 2}
console.log(item); // a

```

如果类数组对象length属性不存在，将添加length属性，并初始化为0.如下：

```js

var o = {0:"a", 1:"b", 2:"c"}
var item = Array.prototype.shift.call(o);
console.log(o); // Object {0:"a", 1:"b", 2: "c", length: 0}
console.log(item); // undefined


```		

#### sort

sort()方法对数组元素进行排序，并返回这个数组。sort方法比较复杂，这里我将多花些篇幅来讲这块。

语法：arr.sort([comparefn])

comparefn是可选的，如果省略，数组将按照各自转换为字符串的Unicode（万国码）位点顺序排序，例如“Boy”将排在“apple”之前。当对数字排序的时候，25将排到8之前，因为转换为字符串后，“25”将比“8”靠前。例如：

```js

var array = ["apple","Boy","Cat","dog"];
var array2 = array.sort();
console.log(array); // ["Boy","Cat","apple","dog"]
console.log(array2 == array); // true

array = [10, 1, 3, 20];
var array3 = array.sort();
console.log(array3); // [1, 10, 20, 3]

```

如果指明了comparefn，数组将按照调用该函数的返回值来排序。若a和b是两个将要比较的元素：

*	若 comparefn(a, b) < 0，那么a 将排到 b 前面；
*	若 comparefn(a, b) = 0，那么a 和 b 相对位置不变；
*	若 comparefn(a, b) > 0，那么a , b 将调换位置；	

如果数组元素为数字，则排序函数comparefn格式如下所示：

```js

function compare(a, b){
	return a-b;
}

```

如果十足元素为非ASCII字符的字符串（如包含类似 e、é、è、a、ä 或中文字符等非英文字符的字符串），则需要使用String.localeCompare。下面这个函数将排到正确的顺序。

```js

var array = ['互','联','网','改','变','世','界'];
var array2 = array.sort();
var array = ['互','联','网','改','变','世','界']; // 重新赋值,避免干扰array2
var array3 = array.sort(function (a, b) {
  return a.localeCompare(b);
});
console.log(array2); // ["世", "互", "变", "改", "界", "网", "联"]
console.log(array3); // ["变", "改", "互", "界", "联", "世", "网"]

```

如上，『互联网改变世界』这个数组，sort函数默认按照数组元素unicode字符串形式进行排序，然而实际上，我们期望的是按照拼音先后顺序进行排序，显然String.localeCompare 帮助我们达到了这个目的。

为什么上面测试中需要重新给array赋值呢，这是因为sort每次排序时改变的是数组本身，并且返回数组引用。如果不这么做，经过连续两次排序后，array2 和 array3 将指向同一个数组，最终影响我们测试。array重新赋值后就断开了对原数组的引用。
	
同上，sort一样受益于鸭式辨型，比如：

```js

var o = {0:'互',1:'联',2:'网',3:'改',4:'变',5:'世',6:'界',length:7};
Array.prototype.sort.call(o,function(a, b){
  return a.localeCompare(b);
});
console.log(o); // Object {0: "变", 1: "改", 2: "互", 3: "界", 4: "联", 5: "世", 6: "网", length: 7}, 可见同上述排序结果一致

```	

注意：使用sort的鸭式辨型特性时，若类数组对象不具有length属性，它并不会进行排序，也不会为其添加length属性。					

```js

var o = {0:'互',1:'联',2:'网',3:'改',4:'变',5:'世',6:'界'};
Array.prototype.sort.call(o,function(a, b){
  return a.localeCompare(b);
});
console.log(o); // Object {0: "互", 1: "联", 2: "网", 3: "改", 4: "变", 5: "世", 6: "界"}, 可见并未添加length属性


```

#### 使用映射改善排序

comparefn如果需要对数组元素多次转换以实现排序，那么使用map辅助排序将是个不错的选择。基本思想就是讲数组的每个元素实际比较的值取出来，排序后将数组恢复。

```js

// 需要被排序的数组
var array = ['dog', 'Cat', 'Boy', 'apple'];
// 对需要排序的数字和位置的临时存储
var mapped = array.map(function(el, i) {
  return { index: i, value: el.toLowerCase() };
})
// 按照多个值排序数组
mapped.sort(function(a, b) {
  return +(a.value > b.value) || +(a.value === b.value) - 1;
});
// 根据索引得到排序的结果
var result = mapped.map(function(el){
  return array[el.index];
});
console.log(result); // ["apple", "Boy", "Cat", "dog"]

```

#### 奇怪的chrome

实际上，ECMAscript规范中并未规定具体的sort算法，这就势必导致各个浏览器不尽相同的sort算法，请看sort方法在Chrome浏览器下的表现：

```js

var array = [{ n: "a", v: 1 }, { n: "b", v: 1 }, { n: "c", v: 1 }, { n: "d", v: 1 }, { n: "e", v: 1 }, { n: "f", v: 1 }, { n: "g", v: 1 }, { n: "h", v: 1 }, { n: "i", v: 1 }, { n: "j", v: 1 }, { n: "k", v: 1 }, ];
array.sort(function (a, b) {
	return a.v - b.v;
});
for (var i = 0,len = array.length; i < len; i++) {
    console.log(array[i].n);
}
// f a c d e b g h i j k

```

由于v值相等，array数组排序前后应该不变，然而Chrome却表现异常，而其他浏览器(如IE 或 Firefox) 表现正常。

```js

var array = [{ n: "a", v: 1 }, { n: "b", v: 1 }, { n: "c", v: 1 }, { n: "d", v: 1 }, { n: "e", v: 1 }, { n: "f", v: 1 }, { n: "g", v: 1 }, { n: "h", v: 1 }, { n: "i", v: 1 }, { n: "j", v: 1 },];
array.sort(function (a, b) {
  return a.v - b.v;
});
for (var i = 0,len = array.length; i < len; i++) {
  console.log(array[i].n);
}
// a b c d e f g h i j

```

从a到j刚好10条数据

那么我么你该如何规避Chrome浏览器的这种“bug”呢？其实很简单，只需略动手脚，改变排序方法的返回值即可，如下：

```js

arr.sort(function (a, b) {
	return a.v - b.v || array.indexOf(a)-array.indexOf(b);
})

```

是数组烦人sort方法需要注意一点各个浏览器针对sort方法内部算法不尽相同，排序函数尽量返回-1,0,1三种不同的值，不要尝试返回true和false等其他数值，天威可能导致不可靠的排序结果。

#### 问题分析

sort方法传入的排序函数如果返回布尔值会导致什么样的结果呢？

一下是常见的浏览器以及脚本引擎：

<table>
	<tr>
		<td>Browser Name</td>
		<td>ECMAScript Engine</td>
	</tr>
	<tr>
		<td>Internet Explorer 6 - 8</td>
		<td>JScript</td>
	</tr>
	<tr>
		<td>Internet Explorer 9 - 10</td>
		<td>Chakra</td>
	</tr>
	<tr>
		<td>Firefox</td>
		<td>SpiderMonkey, IonMonkey, TraceMonkey</td>
	</tr>
	<tr>
		<td>Chrome</td>
		<td>V8</td>
	</tr>
	<tr>
		<td>Safair</td>
		<td>JavaScriptCore(SquirrelFish Extreme)</td>
	</tr>
	<tr>
		<td>Opera</td>
		<td>	Carakan</td>
	</tr>
</table>

分析一下代码，语气将数组元素进行升序排序：

```js

var array = [7,6,5,4,3,2,1,0,8,9];
var comparefn = function (x,y) {
	return x > y;
};
array.sort(comparefn);

```

代码中，comparefn函数返回值为bool类型，并非为规范规定的-1,0,1值。那么执行代码，各JS脚本引擎实现情况如何？

<table>
	<tr>
		<td></td>
		<td>输出结果</td>
		<td>是否符合预期</td>
	</tr>
	<tr>
		<td>JScript</td>
		<td>[2, 3, 5, 1, 4, 6, 7, 0, 8, 9]</td>
		<td>否</td>
	</tr>
	<tr>
		<td>Carakan/td>
		<td>[0, 1, 3, 8, 2, 4, 9, 5, 6, 7]</td>
		<td>否</td>
	</tr>
	<tr>
		<td>Chakra & JavaScriptCore</td>
		<td>[7, 6, 5, 4, 3, 2, 1, 0, 8, 9]</td>
		<td>否</td>
	</tr>
	<tr>
		<td>SpiderMonkey</td>
		<td>[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]</td>
		<td>是</td>
	</tr>
	<tr>
		<td>V8</td>
		<td>[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]</td>
		<td>是</td>
	</tr>
</table>

### 根据表中数据可见，当数组内元素数小于等于10时，现象如下：

*	JScript&Carakan排序结果有误
*	Chakra&JavaScriptCore看起来没有进行排序
*	SpliderMonkey返回了息预期正确结果
*	V8暂时看起来排序正确

### 将数组元素扩大至11位，现象如下：

```js

var array = [7, 6, 5, 4, 3, 2, 1, 0, 10, 9, 8];
var comparefn = function (x, y) {
  return x > y;
};
array.sort(comparefn);

```

<table>
	<tr>
		<td>JavaScript引擎</td>
		<td>输出结果</td>
		<td>是否符合预期</td>
	</tr>
	<tr>
		<td>JScript</td>
		<td>[2, 3, 5, 1, 4, 6, 7, 0, 8, 9, 10]</td>
		<td>否</td>
	</tr>
	<tr>
		<td>Carakan/td>
		<td>[0, 1, 3, 8, 2, 4, 9, 5, 10, 6, 7]</td>
		<td>否</td>
	</tr>
	<tr>
		<td>Chakra & JavaScriptCore</td>
		<td>[7, 6, 5, 4, 3, 2, 1, 0, 10, 8, 9]</td>
		<td>否</td>
	</tr>
	<tr>
		<td>SpiderMonkey</td>
		<td>[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10]</td>
		<td>是</td>
	</tr>
	<tr>
		<td>V8</td>
		<td>[5, 0, 1, 2, 3, 4, 6, 7, 8, 9, 10]</td>
		<td>否</td>
	</tr>
</table>

根据表中数据可见，当数组元素个数大于10时：

*	JScript&Carakan排序结果有误
*	Chakra&JavaScriptCore看起来没有进行排序
*	SpliderMonkey返回预期的正确结果
*	V8**排序结果由正确转为不正确**

### splice

splice()方法用新元素替换就元素的方式来修改数组。它是一个常用的方法，复杂的数组操作场景经常会有它的身影，特别是需要维持元素组引用时，就地删除或者新增元素，splice是最适合的。

语法：arr.splice(start,deleteCount[,item1[,item2[,...]]])

start指定从哪一位开始修改内容。如果唱过了数组长度，则从数组末尾开始添加内容；如果是负值，则其指定的索引位置等同于length+start（length为数组的长度），表示从数组末尾开始的第start位。

deleteCount指定要删除的元素个数，若等于0，则不删除。这种情况下，至少应该添加以为新元素，若大于start之后的元素总和，则start及之后的元素将被删除。

itemN指定新增元素，如果缺省，则该方法只删除数组元素。

返回值由数组中被删除元素组成的数组，如果没有删除，则返回一个空数组

下面来举例子：

```js

var array = ["apple","boy"];
var splices = array.splice(1,1);
console.log(array); // ["apple"]
console.log(splices); // ["boy"] ,可见是从数组下标为1的元素开始删除,并且删除一个元素,由于itemN缺省,故此时该方法只删除元素
array = ["apple","boy"];
splices = array.splice(2,1,"cat");
console.log(array); // ["apple", "boy", "cat"]
console.log(splices); // [], 可见由于start超过数组长度,此时从数组末尾开始添加元素,并且原数组不会发生删除行为
array = ["apple","boy"];
splices = array.splice(-2,1,"cat");
console.log(array); // ["cat", "boy"]
console.log(splices); // ["apple"], 可见当start为负值时,是从数组末尾开始的第-start位开始删除,删除一个元素,并且从此处插入了一个元素
array = ["apple","boy"];
splices = array.splice(-3,1,"cat");
console.log(array); // ["cat", "boy"]
console.log(splices); // ["apple"], 可见即使-start超出数组长度,数组默认从首位开始删除
array = ["apple","boy"];
splices = array.splice(0,3,"cat");
console.log(array); // ["cat"]
console.log(splices); // ["apple", "boy"], 可见当deleteCount大于数组start之后的元素总和时,start及之后的元素都将被删除

```

同上splice一样受益于鸭式辩型，比如：

```js

var o = {0:"apple",1:"boy",length:2};
var splices = Array.prototype.splice.call(o,1,1);
console.log(o); // Object {0: "apple", length: 1}, 可见对象o删除了一个属性,并且length-1
console.log(splices); // ["boy"]

```

注意：如果类数组对象没有length属性，splice将为该类数组对象添加length属性，并初始化为0.

如果需要删除数组中一个已存在的元素，可参考如下：

```js

var array = ['a','b','c'];
array.splice(array.indexOf('b'),1);

```






