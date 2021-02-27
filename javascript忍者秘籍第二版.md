
---

[TOC]

---

# javascript忍者秘籍第二版note
## chapter 6 - 未来的函数:生成器与 promise
generator是特殊类型的函数
promise对象是一个占位符，暂时替代那些尚未计算出但未来会计算出的值

### Question
1. generator 函数的主要用途是什么?

2. 在异步代码中,为什么使用 promise 比使用简单的回调函数更好?

3. 使用 Promise.race 来执行很多长期执行的任务时，promise 最终会在什么时候变成 resolved 状态? 它什么时候会无法变成 resolved 状态?

### notes
1. 原因: 从服务器获取数据是一个长时间操作，js依赖单线程执行模型，所以UI渲染会暂停，应用会无响应。

2. generator
```js
function* WeaponGenerator(){//在关键字function后添加星号*定义生成器函数
	...
	yield "Katana";//在生成器函数内使用yield生成独立的值
	...
}
```

3. 调用生成器不会执行生成器函数，而是创建一个迭代器(iterator)对象与生成器通信

4. 通过迭代器对象控制生成器:
+ 迭代器用于控制生成器的生成。
+ 最基本接口:next 方法，用来向生成器请求一个值。
+ 流程: 生成一个当前值后，生成器就会非阻塞地挂起执行，等待下一次值请求,返回一个对象。属性value为设定值，属性done为false。当没有代码可以执行使，value-undefined done-true

5. 使用生成器
+ 可以无限循环，因为只有请求.next才会迭代下一次的循环
+ 场景:为每个对象赋唯一id值
+ 一般与 for-of 配套
```js
//用生成器遍历DOM树
function* DomTraversal(element){
	yield element;
	element=element.firstElementChild;
	while(element){
		yield* DomTraversal(element); //用 yield* 将迭代控制转移到另一个生成器实例上
		element=element.nextElementSibling;
	}
}

const subTree=document.getElementById("subTree");
for(let element of DomTraversal(subTree)){   //使用 for-of 对节点进行循环迭代
	assert(element!==null,element.nodeName);
}
```

6. 与生成器交互
+ 作为生成器函数参数发送值 & 使用 next 方法向生成器发送值
+ next 的参数传入需要有一个挂起的生成器

7. 探索生成器内部构成
+ 过程:挂起开始 - 执行 - 挂起让渡 - 完成
+ 生成器的执行环境上下文会暂时挂起来并在将来恢复，因为迭代器保持着对其的引用，所以即使 generator 的执行环境从上下文中弹出也不会被销毁

8. 使用promise
+ 处理异步
+ 在promise对象上使用 then 方法，可以传入两个回调函数，调用 resolve 就会调用第一个，而调用 reject 时会调用第二个

9. 简单理解回调函数所带来的问题
+ 回调函数发生错误时无法用内置构造函数处理，错误难以处理
原因: 调用回调函数的代码一般不会和开始任务的这段代码位于事件循环的同意步骤，错误会丢失
+ 执行连续步骤很棘手
+ 执行很多并行任务也棘手

10. 深入研究 promise
+ 一开始是等待状态，然后根据resolve和reject分化为fulfilled和rejected状态
+ then 方法用于建立一个预计在promise被成功实现后执行的回调函数

11. 拒绝 promise
+ 显示拒绝：在一个 promise 的执行函数中调用传入的 reject 方法
+ 隐式拒绝：正处理的一个 promise 的过程中抛出了一个异常，此时会自定位到拒绝回调函数中

12. 把生成器和 promise 相结合
+ 异步任务放入一个生成器中，执行生成器函数，将执行权让渡给生成器，防止阻塞。承诺兑现时通过迭代器的next执行生成器

13. 面向未来的 async 函数
+ async & await

## chapter 7 - 面向对象与原型
+ 探讨原型
+ 使用函数作为构造器
+ 使用原型扩展对象
+ 避免常见的问题
+ 使用继承创建类

### Question
+ 如何知道一个对象是否可以访问特定的属性
+ 在 Javascript 中使用对象时，为什么原型链至关重要
+ ES6 中的关键字 class 是否改变了 Javascript 中对象的工作机制

### notes
1. 理解原型
+ 查找属性时，若对象本身不具有该属性，则会查找原型上是否有该属性
+ 操作符 in 测试对象是否具有某一个特点的属性
+ 对象的原型属性是内置属性，无法直接访问
+ Object.setPrototypeOf(yoshi,hattori),将yoshi的原型设置为hattori

2. 对象构造器与原型
+ 每个函数都有一个原型对象;每个函数的原型都有一个constructor属性，该属性指向函数本身;constructor对象的原型设置为新创建的对象的原型
+ 使用构造 new 时，新构造的对象设置为构造函数的原型的引用

3. 实例属性
+ 在实例中可以查找到的属性，将不会查找原型

4. Javascript 动态特性的副作用
+ 替换构造器原型后，新创建的实例所引用的是替换后的原型

5. 通过构造函数实现对象类型
+ 使用 constructor 属性，我们可以访问创建该对象时所用的函数。这个特性可以用于类型校验
+ 操作符 instanceof 提供了一种用于检测一个实例是否由特定构造函数创建的方法
+ 使用 constructor 的引用创建新对象
```js
function Ninjia(){}

const ninja = new Ninja();
const ninja2 = new ninjia.constructor();
//ninjia 和 ninjia2 是对同一原型的两个不同实例
```

6. 实现继承
```js
Ninja.prototype = new Person();
//实现 Ninja 继承 Person
```
+ 通过 instanceof 可以判定函数是否继承原型链上的对象功能

7. 重写 constructor 属性的问题
+ 设置 Person 实例对象作为 Ninja 构造器的原型时，我们丢失了 Ninja 与 Ninja初始原型之间的关联
+ 调整属性的配置信息，可以使用内置的 Object.defineProperty

8. instanceof 操作符
+ 真正语义：检查右边的函数原型是否存在于操作符左边的对象的原型链上

9. 在 ES6 使用 Javascript 的 class
+ class 是一个语法糖，用于继承，底层是基于原型的实现
+ 静态方法:实例不能访问，类可以访问

10. 实现继承
(ES6)
+ `extends` 实现继承
+ `super` 调用基类构造函数

## chapter 8 - 控制对象的访问
### Question
1. 通过 getter 和 setter 访问属性值有什么好处
2. 代理与 getter 和 setter 的主要区别是什么
3. 代理对象的常见问题是什么？列举3项代理对象的常见问题

### notes
1. 使用 getter 和 setter 控制属性访问
2. 定义 getter 和 setter
+ 通过对象字面量/ES6中的class
+ 使用内置 Object.defineProperty 方法定义
+ 属性名前添加关键字 get/set
+ 获取属性值隐式调用 get，对属性赋值隐式调用 set
+ getter 可以定义计算属性 setter 可用于实现数据验证与日志记录
3. 使用代理控制访问
+ 控制对象的全部交互类型
+ 定义当对象发生交互时可执行的自定义行为，如 读取/设置属性值，或调用方法
+ setter/getter 仅能控制单个对象属性，代理可用于对象交互的通用处理
+ 代理更易于在对象上添加日志

4. 使用代理检测性能
5. 使用代理自动填充属性
6. 使用代理实现负数组索引
7. 代理的性能消耗

## chapter 9 - 处理集合
### Question
1. 使用对象作为字典或 Map 的常见缺陷有哪些?
2. 在 Map 中，键值对可以是哪些类型?
3. Set 中的成员必须是相同类型吗?

### notes
+ 使用 Map 可以创建字典类型，建立键值对的映射关系
+ Set 集合中的成员都是唯一的,不允许出现重复的成员

1. 创建数组
+ 使用内置 Array 构造函数
+ 使用数组字面量 [] 
+ 优先使用数组字面量:
[] 2个字符,由于 js 的高度动态特性,无法阻止修改内置的 Array 构造函数，意味着 new Array() 创建的不一定是数组
+ 数组是对象

2. 在数组两端添加，删除元素
+ push: 末尾
+ unshift: 开头添加
+ pop: 末尾
+ shift: 开头删除
+ pop/push 因为对末尾进行操作，速度会比 shift/unshift 快很多，非特殊情况不使用后者

3. 在数组任意位置添加，删除元素
+ delete: 只是创建空元素
+ splice()

4. 数组常用操作
+ 遍历数组：
```js
ninjas.forEach(ninja=>{
	assert(ninja !== null,ninja);
})
```
+ 基于现有的数组元素映射创建新数组
```js
ninjas.map(ninja=>ninja.weapon);
```
+ 验证数组元素是否匹配指定的条件
```js
ninjas.every(ninja => "name" in ninja);//所有
ninjas.some(ninja => "weapon" in ninja);//至少一个
```
+ 查找特定数组元素
find(ES6):返回满足条件的第一个元素
filter: 返回多个、
indexOf()
lastIndexOf()
findIndex()
+ 聚合数组，基于数组元素计算
.sort()
.reduce()

5. 复用内置的数组函数
模拟类数组方法
```js
function(elem){
	Array.prototype.push.call(this,elem);
}
```
通常 Array.prototype.push 方法通过自身函数上下文执行数组。通过 call() 将上下文改为我们定义的对象

6. 别把对象当作 Map
+ 原型继承属性(存在访问原型属性的风险)以及 Key 仅支持字符串，所以通常不能使用对象作为 map
+ 处理 Map 的一个基本概念是确定两个映射的 key 是否相等
+ forof 循环遍历 Map

7. Set
+ 成员值唯一
+ 并集
```js
new Set([...ninjas],[...samurai]);
```
+ 交集
```js
new Set(
	[...ninjas].filter(ninja => samurai.has(ninja))
	);
```
+ 差集
```js
new Set(
	[...ninjas].filter(ninja => !samurai.has(ninja))//只保留存在于 ninjas 中但不存在与 samurai 中的元素
	);
```