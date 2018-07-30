#JavaScript 原型与原型链

TANGiMING

2018-7-30

=======
# JavaScript 原型与原型链

- 原型是一个对象
- 其他对象可以通过它实现属性继承

### prototype和 \_\_proto\_\_  ###



**prototype**

prototype是function才有的属性,它指向函数的原型对象，且Person.prototype本身就是一个原型对象。

**原型对象（Person.prototype）是 构造函数（Person）的一个实例，它是一个普通对象**

默认情况下，所有的**原型对象**都会自动获得一个constructor(构造函数)属性，这个属性（是一个指针）指向prototype属性所在的函数:

	Person.prototype.constructor==Person //true

**\_\_proto\_\_**


\_\_proto\_\_是每个对象都有的属性

**\_\_proto\_\_是用于指向创建它的构造函数的原型对象**

即：
	
	var a = new function();
	a.__proto__ == Function.prototype //true
	

------------------------------------------------------------------------------	

    var a = {};
    console.log(a.prototype);  //undefined
    console.log(a.__proto__);  //Object {}
    
    var b = function(){}
    console.log(b.prototype);  //b {}
    console.log(b.__proto__);  //function() {}

对象分为两种：**普通对象**和**函数对象**

**普通对象**：


- 最普通对象：有\_\_proto\_\_属性（指向其原型链），没有prototype属性



- 原型对象:(person.prototype 原型对象还有constructor属性（指向构造函数对象）)


**函数对象**：

- 凡是通过new Function()创建的都是函数对象(构造函数)

- 包括Function和Object也是函数对象



- 拥有\_\_proto\_\_、prototype属性（指向原型对象）


**构造函数**

通过new function构造出来的函数成为构造函数

构造函数的实例都有一个constructor属性，这是一个指针，指向构造函数

即：实例的构造函数属性(constructor)指向构造函数:

	var person1 = new Person();
	person.constructor == Person //true


Tips:

由原型对象部分可知：Person.prototype.constructor == Person;
由构造函数默认属性可知：person1.constructor == Person;

可得：person1为Person的一个实例，且Person.prototype也为Person的一个实例

**但是！**

Function.prototype比较特殊，为一个函数对象。

为什么？

由上面可得:

	var person1=new Person();

此时person1由构造函数创建，为一个函数对象，也就是说：

凡事通过new Function()创建的对象都是函数对象，Function.prototype是通过new Function()创建的，所以它是一个函数对象

例：

	person1.__proto__ === Person.prototype;
	//person1.__proto__ === person1的构造函数的prototype

	Person.__proto__ === Function.prototype;
	//同上，Person.__proto__ === Person的构造函数的prototype

	Person.prototype.__proto__ === Object.prototype
	//Person.prototype 是一个普通对象，我们无需关注它有哪些属性，只要记住它是一个普通对象,因为一个普通对象的构造函数 === Object,所以 Person.prototype.__proto__ === Object.prototype

	Object.__proto__ === Function.prototype;
	//参照例2，因为 Person 和 Object 一样都是构造函数

	Object.prototype.__proto__ === null
	//Object.prototype 对象也有proto属性，但它比较特殊，为 null 。因为 null 处于原型链的顶端

需要记住的是，Function的prototype和__proto__:

**Function.prototype为一个空函数（Empty function），**

**Function.\_\_proto\_\_ === null;**

为什么？

因为**所有的构造器都来自于 Function.prototype，甚至包括根构造器Object及Function自身。所有构造器都继承了Function.prototype的属性及方法。如length、call、apply、bind**

**原型对象的主要作用为继承:**
	
    function test(){
    	this.value=40;
    }
	var result=new test();
	console.log(result.value);//40
	//函数执行时,this指向result
	//new出来的result对象继承了test的prototype原型
	function foo(){};
	foo.prototype=new test();
	foo.prototype.word="Hello World!";
	console.log(foo.prototype); //value:40,word:"Hello World!"


	test.prototype={
		add:function(a,b){
			return a+b;
		},
		subtract:function(a,b){
			return a-b;		}
	}
	var result2=new test();
	console.log(result.add); //undefined
	console.log(result2.value); //40
	console.log(result2.add); //function(a,b) {return a+b};
	console.log(result2.subtract(3,2)); //1

	//向test添加了两个方法，new出来的result2继承了test的所有属性和方法


**原型链：**

实例化的对象在进行属性查找时，如果没有查找到这个属性或者方法，会逐级向上查找，直到查找到Object的原型上，这一个逐级向上查找的链，就是原型链：

	Object.prototype.bar=1;
	function test2(){
		this.val=10;
	}
	var result3=new test2();
	console.log(result3.val); //10
	console.log(result3.bar); //1
	//result3会依据原型链逐级向上查找存在该属性的原型，直到找到，否则为undefined
