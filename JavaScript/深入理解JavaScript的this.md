### 深入理解JavaScript的this

###### @copyRight shenyong 2018-08-31

#####  1.什么是this?

this关键字想必大家都不陌生，简单来说`this`就是指向我们调用函数的执行上下文。

```
执行上下文 是语言规范中的一个概念，用通俗的话讲，大致等同于函数的执行“环境”。具体的有：变量作用域（和 作用域链条，闭包里面来自外部作用域的变量），函数参数，以及 this 对象的值。
```

那么问题来了，函数的执行环境到底怎么分辨呢？是什么调用了函数？是哪个对象调用了函数？

我们可以看如下例子

```
var person = {
    name: "Jay",
    greet: function() {
        console.log("hello, " + this.name);
    }
};
person.greet();
```

在上面例子中创建了一个叫`person`的对象，然后`person`这个对象调用了函数`greet()`那么在函数`greet`中this关键字指向的就是`person` ,`this.name`就等于`"Jay"`

那如果把上面的例子稍微改一下

```
var greet = person.greet;//创建一个对象
greet();//调用函数
```

那么在这种情况下会打印出什么呢？我们可以看到最终调用函数的上下文是`greet`，那么`greet`对象里面是没有`name`这个变量的,所以打印出来的是`undefined`。

* 所以说`this`的值并不是由函数放在哪个对象里面决定的，而是函数执行时由谁唤起决定的。

##### 2.this的指向

从上面的列子可以看出this所在的位置不同那么其指向的对象也会有变化，我们先看下面的例子

```
var name = "Jay Global";
var person = {
    name: 'Jay Person',
    details: {
        name: 'Jay Details',
        print: function() {
            return this.name;
        }
    },
    print: function() {
        return this.name;
    }
};
console.log(person.details.print());  // ?
console.log(person.print());          // ?
var name1 = person.print;
var name2 = person.details;
console.log(name1()); // ?
console.log(name2.print()) // ?
```

 #####  解析：

* `person.details.print()` 这这里到底是谁调用了`print`函数呢？在JavaScript中我们都是从左读到右。于是这里调用函数的是在`person`对象里面的`detail`, 所以在`print`里面的`this`指向的不是`person`而是`details` 。那么函数输出的就是`Jay Details`
* 在`person.print()`里面调用函数的是`person`那么打印的就是输入的就是`Jay Person`。
* `var name1 = person.print` 会让人误以为调用环境是`person`，但最终调用的其实是`name1()`，而`this`关键字实在函数调用时才做绑定，`name1`前面什么也没有，因此`this`关键字指向全局的`window`对象，所以输出的是`Jay Global`
* name2指向details对象，所以会输出`Jay Details`

##### 总结

只有当定义`this`的函数被对象调用时，`this`才会被赋值。当代码在浏览器里执行时，全局作用域里的所有全局变量和函数都在`window`对象里定义，所以在全局函数里使用`this`，它指代window对象并储存着该对象的值。window对象是整个JavaScript程序或网页的主储存器。在闭包函数或者回调函数里面使用`this`要格外谨慎,因为在闭包使用`this`关键词无法访问外部函数的`this`变量。这种情况想要使用外部函数的变量可以通过`let _this = this` 用`_this`来代替this，这样就不会混淆。



















