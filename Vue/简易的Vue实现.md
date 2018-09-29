# 简易的 Vue 实现

TANGIMING

2018/9/29 14:07:17

**Vue 是一个声明式的（配置式）的前端 MVVM 框架，可以让视图上的数据与交互逻辑中的数据双向绑定，实时的更新视图，解决了传统前端开发中需要频繁获取 dom 的操作**

## Vue 是怎么运作的？它又是如何实现的？

Vue 主要的双向绑定是通过 ES5 中，Object.definedProperty()这个方法，为对象属性设置一个 getter 和 setter，让其每次发生变化时都可以实时的拦截并且做出相应的处理，由于这个方法只在 ES5 才被支持，所以使用 Vue 有一定的版本限制（IE9+）

如果你不知道 ES5 和 Object.definedProperty()这个方法，你应该努努力了！

### 如何实现

谈及 Vue，主要就有两点：

1. 虚拟 DOM

虚拟 DOM 其实说来简单，实现起来确比较有难度。虚拟 DOM 的原理为在真实的 DOM 树被插入根结点之前，转换为字符串进行一系列的操作（渲染数据，操作 DOM 等），等处理完毕之后再插入根结点，这样就把传统的 DOM 操作转换成为了对字符串的操作。性能可以得到极大的提升；

2. 双向绑定

如开头所说，利用 Object.definedProperty()来对数据设置一个 getter 和 setter,来实现数据和视图的同步更新；

鉴于虚拟 DOM 的工作量比较大，涉及大量的正则和字符串操作，所以本文没有关于实现虚拟 DOM 的方法，只讨论双向绑定；

**下面让我们来实现一个简易的 Vue，来加深对 Vue 内部的理解**

运行过程：

Vue 初始化 -> 数据代理 -> 绑定事件方法 -> 创建监听任务的列表 -> 开始监听数据 -> 解析 dom

我们采用 ES6 中 class 的方法来编写，这样更加清晰的能够表达结构和关系：

首先，新建一个 class：

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

这大概是从初始化到完成数据渲染到 dom 的整个过程，我们先写好流程，再去完善每一个方法；

首先，当我们 new Vue 的时候，就会自动调用构造方法 constructor()

然后查找到我们注入节点的 dom，接着在实例化的对象取得 data 对象，并且进行数据代理；

接着我们后面再讲，先来看数据代理：

数据代理的作用是将 this.data 里面的数据代理到 this 上，因为我们在使用 Vue 的过程中，访问某个数据是通过 this.xxx 而不是 this.data.xxx，所以我们需要将 data 里面的数据挂载到 this 上去：

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

先将 data 里面所有的数据的 key 拿出来，对每一个 key 调用数据代理的方法；
然后对这个属性设置 getter 和 setter，每一次获取值得时候是直接得到的 this.data.xxx 的值，改变值的时候也是直接改变的 this.data.xxx 的值，这样就完成了将 this.data 里面的数据映射到 this 上；

接着，将我们写在 methods 里面方法直接放在 this.methods 上：

    this.methods = opt.methods;

然后，我们新建了一个对象：

    //需要监听的任务列表
    this.watcherTask = {};

这样做的目的在于，我想要吧初始化时的对象一起塞进 watcherTask 里面，这样我在更新试图的时候，就可以一次性全部更新，而不是去一个一个更新:

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

同样的先将所有的数据放进 watcherTask 中，然后对所有数据设置 Getter 和 Setter，当获取值得时候，直接返回 data 里面的值，当设置值的时候，除了要改变值之外，还要对 watcherTask 里面的所有值进行更新，因为我并不知道用户到底改变了哪些值，所以我需要全部更新；

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

由于没有使用虚拟 dom，所以我这里换了一种解决办法，先获取到根节点之后，递归的查找该节点下的所有子节点，这一步的主要目的是解析节点中的双花括号（ {{ }} ），我们可以看到，我对每一个解析到的节点都调用了 compileText 方法，以下是该方法:

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

除了捕获花括号之外，还要依次的判断节点上是否存在我的指令（v-model,v-html 等）以及是否监听事件(@click 等)，这都是通过 getAttribute 来获取到的：

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

如果获取到了 v-model 属性，那么我会认为这是一个表单组件（因为 Vue 中 v-model 大部分情况下都是对表单使用），如果你不知道为什么，那么你可以去 Vue 的官网了解一下，v-model 到底是什么。

接着，监听这个表单组件的 input 事件，然后每当 input 事件触发时，就调用 Watcher 方法来更新数据和视图，更新完毕之后，再删除掉 dom 节点上的 v-model 指令（因为你可能不太想别人在控制台里面看到你绑定在 dom 节点上的指令和数据）

v-html,v-text 等指令同理，就不再阐述了

当然，对于事件的监听方法大同小异，得到@click 这个属性之后对其绑定 click 事件并且绑定 this，这样你就可以在方法内部调用 this 了；

最后，我们来看看当数据更改时，视图是如何更新。你可能发现了，在上面的代码中得到的指令后，都会实例化一个 Watcher，那么这个 Watcher 实际上就是用来更新视图的：

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

很简单，Wacher 初始化之后调用 update 方法，然后将挂载在根节点里面的所有数据进行数据更新

到这里，一个基本的 Vue 就开发完成了，你可以通过像使用 Vue 一样使用它：

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

以下是完整的 js 代码：

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
    </body>

    </html>

复制到你的编辑器中，应该就可以使用了。

希望你可以通过这篇文章，来更加了解 Vue！
