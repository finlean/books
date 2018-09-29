# 简易的Vue实现 #

TANGIMING

2018/9/29 14:07:17 

**Vue是一个声明式的（配置式）的前端MVVM框架，可以让视图上的数据与交互逻辑中的数据双向绑定，实时的更新视图，解决了传统前端开发中需要频繁获取dom的操作**

## Vue是怎么运作的？它又是如何实现的？ ##

Vue主要的双向绑定是通过ES5中，Object.definedProperty()这个方法，为对象属性设置一个getter和setter，让其每次发生变化时都可以实时的拦截并且做出相应的处理，由于这个方法只在ES5才被支持，所以使用Vue有一定的版本限制（IE9+）

如果你不知道ES5和Object.definedProperty()这个方法，你应该努努力了！

### 如何实现 ###

谈及Vue，主要就有两点：

1. 虚拟DOM

虚拟DOM其实说来简单，实现起来确比较有难度。虚拟DOM的原理为在真实的DOM树被插入根结点之前，转换为字符串进行一系列的操作（渲染数据，操作DOM等），等处理完毕之后再插入根结点，这样就把传统的DOM操作转换成为了对字符串的操作。性能可以得到极大的提升；

2. 双向绑定

如开头所说，利用Object.definedProperty()来对数据设置一个getter和setter,来实现数据和视图的同步更新；

鉴于虚拟DOM的工作量比较大，涉及大量的正则和字符串操作，所以本文没有关于实现虚拟DOM的方法，只讨论双向绑定；


**下面让我们来实现一个简易的Vue，来加深对Vue内部的理解**

运行过程：

Vue初始化 -> 数据代理 -> 绑定事件方法 -> 创建监听任务的列表 -> 开始监听数据 -> 解析dom

我们采用ES6中class的方法来编写，这样更加清晰的能够表达结构和关系：

首先，新建一个class：

	class Vue {
		//初始化，在实例化的时候会最先执行，opt为传入的参数 
		constructor(opt={}){
			/获取dom入口 -> app
	        this.$el = document.querySelector(opt.el);
	        let data = (this.data = opt.data);
	        //代理data，为了让this.data可以直接访问data
	        Object.keys(data).forEach(val => {
	            this.proxyData(val);
	        });
	        //绑定的事件方法
	        this.methods = opt.methods;
	        //需要监听的任务列表
	        this.watcherTask = {};
	        //初始化监听劫持所有数据
	        this.observer(data);
	        //解析dom
	        this.compile(this.$el);
		}
	}

这大概是从初始化到完成数据渲染到dom的整个过程，我们先写好流程，再去完善每一个方法；

首先，当我们new Vue的时候，就会自动调用构造方法constaructor()

然后查找到我们注入节点的dom，接着在实例化的对象取得data对象，并且进行数据代理；

接着我们后面再讲，先来看数据代理：

数据代理的作用是将this.data里面的数据代理到this上，因为我们在使用Vue的过程中，访问某个数据是通过this.xxx而不是this.data.xxx，所以我们需要将data里面的数据挂载到this上去：

	Object.keys(data).forEach(val => {
            this.proxyData(val);
    });
	//数据代理
    proxyData(key) {
        let that = this;
        //主要是代理 data到最上层， this.xxx的方式直接访问 data而不是this.data.xxx
        Object.defineProperty(that, key, {
            configurable: false,
            enumerable: true,
            get() {
                // 每一次获取值，直接代理到this.data.xxx
                return that.data[key];
            },
            set(newVal) {
                that.data[key] = newVal;
            }
        });
    }

先将data里面所有的数据的key拿出来，对每一个key调用数据代理的方法；
然后对这个属性设置getter和setter，每一次获取值得时候是直接得到的this.data.xxx的值，改变值的时候也是直接改变的this.data.xxx的值，这样就完成了将this.data里面的数据映射到this上；

接着，将我们写在methods里面方法直接放在this上：

	this.methods = opt.methods;

同样的，我们调用方法的时候就是this.xxx()而不是this.methods.xxx()了；

然后，我们新建了一个对象：

	//需要监听的任务列表
    this.watcherTask = {};

这样做的目的在于，我想要吧初始化时的对象一起塞进watcherTask里面，这样我在更新试图的时候，就可以一次性全部更新，而不是去一个一个更新:

	//初始化监听劫持所有数据
    this.observer(data);
	observer(data) {
        let that = this;
        /**
         * 同样使用defineProperty来监听数据，初始化需要订阅的数据，
         * 把需要订阅的数据push到watcherTask里，等到需要更新的时候就可以批量更新数据
         */
        Object.keys(data).forEach(key => {
            let value = data[key];
            this.watcherTask[key] = [];
            Object.defineProperty(data, key, {
                configurable: false,
                enumerable: true,
                get() {
                    return value;
                },
                set(newVal) {
                    if (newVal !== value) {
                        value = newVal;
                        //遍历订阅池，批量更新视图
                        that.watcherTask[key].forEach(task => {
                            task.update();
                        });
                    }
                }
            });
        });
    }

同样的先将所有的数据放进watcherTask中，然后对所有数据设置Getter和Setter，当获取值得时候，直接返回data里面的值，当设置值的时候，除了要改变值之外，还要对watcherTask里面的所有值进行更新，因为我并不知道用户到底改变了哪些值，所以我需要全部更新；

到这里，数据的实时更新的基本内容就完成了，接下来就是重要部分：

    //解析dom
    this.compile(this.$el);

	//解析DOM
    compile(el) {
        var nodes = el.childNodes;
        //遍历根节点下面的所有子节点
        for (let i = 0; i < nodes.length; i++) {
            const node = nodes[i];
            if (node.nodeType === 3) {
                //代表当前是文本节点
                var text = node.textContent.trim();
                if (!text) continue;
                this.compileText(node, 'textContent');
            } else if (node.nodeType === 1) {
                //代表当前是元素节点
                if (node.childNodes.length > 0) {
                    //当然，需要递归去查找所有节点
                    this.compile(node);
                }
                /**
                 * 这里是关于指令的代码
                 * 首先去看节点上是否有指令，如果有的话，就发布订阅：
                 * 将订阅的数据push到watcherTask里，后面就可以一次性批量更新，来实现双向绑定
                 */
                if (
                    node.hasAttribute('v-model') &&
                    (node.tagName === 'INPUT' || node.tagName === 'TEXTAREA')
                ) {
                    node.addEventListener(
                        'input',
                        (() => {
                            let attrVal = node.getAttribute('v-model');
                            this.watcherTask[attrVal].push(
                                new Watchter(node, this, attrVal, 'value')
                            );
                            node.removeAttribute('v-model');
                            return () => {
                                this.data[attrVal] = node.value;
                            };
                        })()
                    );
                }
                if (node.hasAttribute('v-html')) {
                    let attrVal = node.getAttribute('v-html');
                    //这里就是批量更新的例子
                    this.watcherTask[attrVal].push(
                        /**
                         * 这里push进去的是一个watcher实例，因为new的时候会先执行一次
                         * 就会把双花括号的数据更新到节点上了
                         */
                        new Watchter(node, this, attrVal, 'innerHTML')
                    );
                    //当然，还需要把节点上的自定义指令属性给删掉，因为你肯定不想看到节点上还绑定着数据
                    node.removeAttribute('v-html');
                }
                this.compileText(node, 'innerHTML');
                if (node.hasAttribute('@click')) {
                    let attrVal = node.getAttribute('@click');
                    node.removeAttribute('@click');
                    node.addEventListener('click', e => {
                        this.methods[attrVal] &&
                            this.methods[attrVal].bind(this)();
                    });
                }
            }
        }
    }

由于没有使用虚拟dom，所以我这里换了一种解决办法，先获取到根节点之后，递归的查找该节点下的所有子节点，这一步的主要目的是解析节点中的双花括号（ {{ }} ），我们可以看到，我对每一个解析到的节点都调用了compileText方法，以下是该方法:

	//解析DOM里花括号的的操作
    compileText(node, type) {
        let reg = /\{\{(.*)\}\}/g,
            txt = node.textContent;
        if (reg.test(txt)) {
            node.textContent = txt.replace(reg, (metched, value) => {
                let tpl = this.watcherTask[value] || [];
                tpl.push(new Watchter(node, this, value, type));
                return value.split('.').reduce((val, key) => {
                    return this.data[key];
                }, this.$el);
            });
        }
    }

除了捕获花括号之外，还要依次的判断节点上是否存在我的指令（v-model,v-html等）以及是否监听事件(@click等)，这都是通过getAttribute来获取到的：

	if (node.hasAttribute('v-model') && (node.tagName === 'INPUT' || node.tagName === 'TEXTAREA')) {
	    node.addEventListener(
	        'input',
	        (() => {
	            let attrVal = node.getAttribute('v-model');
	            this.watcherTask[attrVal].push(
	                new Watchter(node, this, attrVal, 'value')
	            );
	            node.removeAttribute('v-model');
	            return () => {
	                this.data[attrVal] = node.value;
	            };
	        })()
	    );
	}
	if (node.hasAttribute('v-html')) {
	    let attrVal = node.getAttribute('v-html');
	    //这里就是批量更新的例子
	    this.watcherTask[attrVal].push(
	        /**
	         * 这里push进去的是一个watcher实例，因为new的时候会先执行一次
	         * 就会把双花括号的数据更新到节点上了
	         */
	        new Watchter(node, this, attrVal, 'innerHTML')
	    );
	    //当然，还需要把节点上的自定义指令属性给删掉，因为你肯定不想看到节点上还绑定着数据
	    node.removeAttribute('v-html');
	}
	this.compileText(node, 'innerHTML');
	if (node.hasAttribute('@click')) {
	    let attrVal = node.getAttribute('@click');
	    node.removeAttribute('@click');
	    node.addEventListener('click', e => {
	        this.methods[attrVal] &&
	            this.methods[attrVal].bind(this)();
	    });
	}

如果获取到了v-model属性，那么我会认为这是一个表单组件（因为Vue中v-model大部分情况下都是对表单使用），如果你不知道为什么，那么你可以去Vue的官网了解一下，v-model到底是什么。

接着，监听这个表单组件的input事件，然后每当input事件触发时，就调用Watcher方法来更新数据和视图，更新完毕之后，再删除掉dom节点上的v-model指令（因为你可能不太想别人在控制台里面看到你绑定在dom节点上的指令和数据）

v-html,v-text等指令同理，就不再阐述了

当然，对于事件的监听方法大同小异，得到@click这个属性之后对其绑定click事件并且绑定this，这样你就可以在方法内部调用this了；

最后，我们来看看当数据更改时，视图是如何更新。你可能发现了，在上面的代码中得到的指令后，都会实例化一个Watcher，那么这个Watcher实际上就是用来更新视图的：

	//更新视图的操作
	class Watchter {
	    constructor(el, vm, value, type) {
	        this.el = el;
	        this.vm = vm;
	        this.value = value;
	        this.type = type;
	        this.update();
	    }
	    update() {
	        this.el[this.type] = this.vm.data[this.value];
    }
}

很简单，Wacher初始化之后调用update方法，然后将挂载在根节点里面的所有数据进行数据更新

到这里，一个基本的Vue就开发完成了，你可以通过像使用Vue一样使用它：

	<div id="app">
        <p v-html="test"></p>
        <input type="text" v-model="form">
        <button @click="changeValue">改变值</button>
        {{form}}
    </div>
    <script>
        new Vue({
            el: '#app',
            data: {
                test: '这是测试的test',
                form: '这是表单的val'
            },
            methods: {
                changeValue() {
                    this.test = this.form;
                }
            }
        })
    </script>

以下是完整的js代码：

	class Vue {
    //数据初始化
    constructor(opt = {}) {
        //获取dom入口 -> app
        this.$el = document.querySelector(opt.el);
        let data = (this.data = opt.data);
        //代理data，为了让this.data可以直接访问data
        Object.keys(data).forEach(val => {
            this.proxyData(val);
        });
        //绑定的事件方法
        this.methods = opt.methods;
        //需要监听的任务列表
        this.watcherTask = {};
        //初始化监听劫持所有数据
        this.observer(data);
        //解析dom
        this.compile(this.$el);
    }
    //数据代理
    proxyData(key) {
        let that = this;
        //主要是代理 data到最上层， this.xxx的方式直接访问 data而不是this.data.xxx
        Object.defineProperty(that, key, {
            configurable: false,
            enumerable: true,
            get() {
                // 每一次获取值，直接代理到this.data.xxx
                return that.data[key];
            },
            set(newVal) {
                that.data[key] = newVal;
            }
        });
    }
    //数据变化来监听并且劫持变化数据
    observer(data) {
        let that = this;
        /**
         * 同样使用defineProperty来监听数据，初始化需要订阅的数据，
         * 把需要订阅的数据push到watcherTask里，等到需要更新的时候就可以批量更新数据
         */
        Object.keys(data).forEach(key => {
            let value = data[key];
            this.watcherTask[key] = [];
            Object.defineProperty(data, key, {
                configurable: false,
                enumerable: true,
                get() {
                    return value;
                },
                set(newVal) {
                    if (newVal !== value) {
                        value = newVal;
                        //遍历订阅池，批量更新视图
                        that.watcherTask[key].forEach(task => {
                            task.update();
                        });
                    }
                }
            });
        });
    }
    //解析DOM
    compile(el) {
        var nodes = el.childNodes;
        //遍历根节点下面的所有子节点
        for (let i = 0; i < nodes.length; i++) {
            const node = nodes[i];
            if (node.nodeType === 3) {
                //代表当前是文本节点
                var text = node.textContent.trim();
                if (!text) continue;
                this.compileText(node, 'textContent');
            } else if (node.nodeType === 1) {
                //代表当前是元素节点
                if (node.childNodes.length > 0) {
                    //当然，需要递归去查找所有节点
                    this.compile(node);
                }
                /**
                 * 这里是关于指令的代码
                 * 首先去看节点上是否有指令，如果有的话，就发布订阅：
                 * 将订阅的数据push到watcherTask里，后面就可以一次性批量更新，来实现双向绑定
                 */
                if (
                    node.hasAttribute('v-model') &&
                    (node.tagName === 'INPUT' || node.tagName === 'TEXTAREA')
                ) {
                    node.addEventListener(
                        'input',
                        (() => {
                            let attrVal = node.getAttribute('v-model');
                            this.watcherTask[attrVal].push(
                                new Watchter(node, this, attrVal, 'value')
                            );
                            node.removeAttribute('v-model');
                            return () => {
                                this.data[attrVal] = node.value;
                            };
                        })()
                    );
                }
                if (node.hasAttribute('v-html')) {
                    let attrVal = node.getAttribute('v-html');
                    //这里就是批量更新的例子
                    this.watcherTask[attrVal].push(
                        /**
                         * 这里push进去的是一个watcher实例，因为new的时候会先执行一次
                         * 就会把双花括号的数据更新到节点上了
                         */
                        new Watchter(node, this, attrVal, 'innerHTML')
                    );
                    //当然，还需要把节点上的自定义指令属性给删掉，因为你肯定不想看到节点上还绑定着数据
                    node.removeAttribute('v-html');
                }
                this.compileText(node, 'innerHTML');
                if (node.hasAttribute('@click')) {
                    let attrVal = node.getAttribute('@click');
                    node.removeAttribute('@click');
                    node.addEventListener('click', e => {
                        this.methods[attrVal] &&
                            this.methods[attrVal].bind(this)();
                    });
                }
            }
        }
    }
    //解析DOM里花括号的的操作
	    compileText(node, type) {
	        let reg = /\{\{(.*)\}\}/g,
	            txt = node.textContent;
	        if (reg.test(txt)) {
	            node.textContent = txt.replace(reg, (metched, value) => {
	                let tpl = this.watcherTask[value] || [];
	                tpl.push(new Watchter(node, this, value, type));
	                return value.split('.').reduce((val, key) => {
	                    return this.data[key];
	                }, this.$el);
	            });
	        }
    	}
	}
	//更新视图的操作
	class Watchter {
	    constructor(el, vm, value, type) {
	        this.el = el;
	        this.vm = vm;
	        this.value = value;
	        this.type = type;
	        this.update();
	    }
	    update() {
	        this.el[this.type] = this.vm.data[this.value];
	    }
	}

	
以下是使用时的完整代码：

	<!DOCTYPE html>
	<html lang="en">
	<head>
	    <meta charset="UTF-8">
	    <meta name="viewport" content="width=device-width, initial-scale=1.0">
	    <meta http-equiv="X-UA-Compatible" content="ie=edge">
	    <script src="https://cdn.bootcss.com/babel-polyfill/7.0.0-beta.49/polyfill.min.js"></script>
	    <script src='./Vue.js'></script>
	    <title>Riz</title>
	</head>
	
	<body>
	    <div id="app">
	        <p v-html="test"></p>
	        <input type="text" v-model="form">
	        <button @click="changeValue">改变值</button>
	        {{form}}
	    </div>
	    <script>
	        new Riz({
	            el: '#app',
	            data: {
	                test: '这是测试的test',
	                form: '这是表单的val'
	            },
	            methods: {
	                changeValue() {
	                    this.test = this.form;
	                }
	            }
	        })
	    </script>
	</body>
	
	</html>

复制到你的编辑器中，应该就可以使用了。

希望你可以通过这篇文章，来更加了解Vue！




