# Vue组件化开发模式 #

2018/8/28 15:37:41 TANGiMING

不管是react还是vue，目前前端流行的开发模式总体来讲有两种：

1、直接基于HTML，几乎没有组件的概念，使用jquery等库进行开发，开发速度较快，但复用性和维护性较差；

2、基于webpack等构建方法进行组件化的开发，将项目中的大部分拆分为组件，可靠性较高，有较高的可维护性和复用性；

第一种开发模式属于比较老的开发模式，但受益于webpack等自动化打包、构建的工具，现在使用组件化开发模式已经成为一种主流，再加上Vue、React等MVVM框架加上基于框架生态的UI框架，使前端开发更加便捷，也大幅度降低了前端开发的难度；本文将会重点介绍在Vue中，如何基于Vue进行组件化开发。使用Vue进行组件化开发大体有两种方式：不使用构建工具和使用webpack等构建工具进行开发；


## 不使用构建工具的开发模式 ##

在没有使用构建、打包工具的项目中，一般来讲是直接通过HTML中写CSS和JS，自然也没有单页面（SPA）的说法，在这种情况下基于VUE进行开发的例子如下：

    <!DOCTYPE html>
	<html lang="en">
	<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <script src="https://cdn.bootcss.com/vue/2.5.17-beta.0/vue.min.js"></script>
    <title>DEMO</title>
	</head>
		<style>
		/* code CSS here */
		</style>
		<body>
		    <div id="app"></div>
		    <script>
		        var vm=new Vue({
		            el:'#app',
		            data:{
		
		            },
		            methods: {
		                
		            },
		            created(){
		                
		            }
		        })
		    </script>
		</body>
		</html>

我们通过CDN的方式引入了Vue（或者本地相对路径引入）,在script标签中声明一个Vue实例，并且将我们的数据、方法、生命周期方法作为初始化参数进行传递，唯一需要注意的是，在这种开发方式下面,data为一个对象，而非一个方法。


**那么在这种模式下，我如何去编写我的自定义组件呢？**

由于没有引入webpack以及Babel之类的打包、编译工具，所以我们是不能进行import这种操作，并且在HTML中，也识别不了.vue的后缀文件，所以当我们需要写自定义组件的时候，需要这样去创建

	//js
	Vue.component('myComonent',{
		template:'<div>this components name is {{comName}}</div>',
		data:function(){
			return {
				comName:'myComponent'
			}
		}
	})

这样就可以创建一个组件了。

创建完成后，你可以直接在HTML里面使用了：

	<div id='app'>
		<myComponent></myComponents>
	</div>

其实可以发现，在这种方法下去使用Vue的组件化开发模式是比较繁琐的，加上不能引入.vue的文件，造成一个组件在多个页面里面复用会显得异常的麻烦，所以如果因为某些情况必须使用这种开发模式的话，个人建议将Vue仅仅作为一个双向数据绑定的库，用来绑定数据和写一些方法是比较方便的。

## 使用Webpack等构建工具进行开发 ##

所谓的webpack，它的主要作用是将你项目中互相有引用关系的文件进行打包，并且简化你的代码进行合并，提升项目性能以及减小项目体积，你不需要明白他是如何工作的，在项目开发中，你也许只需要明白他是用来干什么的就足够了，再加上babel等编译工具，可以将一些ES6\7\8的语法转换成浏览器可以识别的ES5语法，可以更加提升我们开发的效率。

在这种模式下，配合Vue可以极大提升开发效率，并且可以使用Vue特有的**单文件组件**

### 什么是单文件组件？ ###

Vue中的单文件组件是该框架特有的一种文件，文件后缀以.vue命名，文件中可以写HTML、CSS、Javascript,与传统的HTML不同的是，它可以在任何js文件，其他的单文件组件中被引入。

单文件组件可以是一个页面，也可以是一个小的组件，一个自定义的按钮，甚至是一个高度封装的列表，由于目前vue项目中绝大部分都是使用的单文件组件进行开发，本文将重点介绍单文件组件的使用，它就是用来完成组件化开发的重要因素。

一个基本的单文件组件结构：

首先，假设你已经使用了vue-cli脚手架创建了一个vue项目，我们就可以直接在src目录下创建一个start.vue文件，它长这样：

	<template>
    	<div class='start'>

		</div>
	</template>

	<script>
	export default {
	    
	}
	</script>
	<style>
	
	</style>

其中template便是写HTML的地方，需要注意的是每一个单文件组件的template标签里面只能有一个div，你自己的HTML必须写在div里面，但是绝对不能和这个div同级，你甚至不需要知道这是为什么，因为这是Vue的规则。

这里我们通过单文件组件来构建一个自定义按钮的组件来演示如何创建一个组件。

该按钮的功能有：自定义按钮文字，点击事件

首先，我们先在HTML中编写这个按钮的结构：

	<template>
    	<div class='start'>
			<div class='btn'></div>
		</div>
	</template>

使用div来模拟按钮而非使用Button标签的目的是让我们有更多的可以自定义

当然，在js中，我们也先需要初始化一些东西：

	<script>
		export default {
		    name:'btn',
		    data () {
		        return {
		            
		        }
		    },
		    props: {
		
		    },
		    methods: {
		        
		    },
		    created () {
		        
		    }
		}
	</script>

CSS的部分我就不写了，你想让他长啥样就长啥样吧

现在我们需要来实现第一个功能——自定义按钮文字：

在这之前，我们先实现第一步，让按钮的文字为动态的：

	<template>
    	<div class='start'>
			<div class='btn'>{{btnText}}</div>
		</div>
	</template>
	<script>
		export default {
		    name:'btn',
		    data () {
		        return {
		            btnText:'这是按钮的文字'
		        }
		    },
		    props: {
		
		    },
		    methods: {
		        
		    },
		    created () {
		        
		    }
		}
	</script>

通过在data中声明一个btnText属性，并且将其绑定到HTML上，这样当btnText的值改变时，HTML上的值也会随之改变，你甚至可以在created（该组件被初始化）的时候设置它的值:


	<template>
    	<div class='start'>
			<div class='btn'>{{btnText}}</div>
		</div>
	</template>
	<script>
		export default {
		    name:'btn',
		    data () {
		        return {
		            btnText:'这是按钮的文字'
		        }
		    },
		    props: {
		
		    },
		    methods: {
		        
		    },
		    created () {
		        this.btnText='这是在初始化时按钮的值'
		    }
		}
	</script>

这样，就可以动态改变按钮的文字了。那么现在我们需要让其可以在使用时自定义，该组件在其他地方被使用时，它一定是这样的

	//app.vue
	<template>
    	<div class='index-app'>
			//这里就可以使用了
			<btn></btn>
		</div>
	</template>
	<script>
	//使用组件必须先引入
	import btn from './btn.vue'
		export default {
			//并且需要在这里进行声明注册这个组件,这里两步是必须的
			components:{
				btn
			}
		}
	</script>

那么我们是怎样自定义文字呢？这就需要一个Vue的功能，叫做props，props是用来父组件来给子组件传递参数的，在这里，app.vue（页面）属于父组件，btn.vue属于子组件。

首先，需要在子组件内声明props,由于按钮文字是从外部传递进来，所以data里面的btnText也不需要了：

	//btn.vue
	<template>
    	<div class='start'>
			<div class='btn'>{{btnText}}</div>
		</div>
	</template>
	<script>
		export default {
		    name:'btn',
		    data () {
		        return {
		        }
		    },
		    props: {
				//声明btnText
				btnText:{
					type:String,//声明该属性为一个字符串
					default:'点击'//默认值为 点击,当外部没有传递时，将会采用这个值
				}
		    },
		    methods: {
		        
		    },
		    created () {
		    }
		}
	</script>

现在我们就可以在父组件传递我们想要他展示的文字了：

	
	//app.vue
	<template>
    	<div class='index-app'>
			//这里来标记传递的数据为btnText
			<btn :btnText='btnText'></btn>
		</div>
	</template>
	<script>
	//使用组件必须先引入
	import btn from './btn.vue'
		export default {
			//并且需要在这里进行声明注册这个组件,这里两步是必须的
			components:{
				btn
			},
			data(){
				return {
					btnText:'点一下'//这里来表示传入的文字
				}
			}
		}
	</script>

现在，自定义文字的功能就已经完成了。

现在我们继续来开发另一个功能，按钮的点击事件，首先，需要在子组件内：

	
	
	//btn.vue
	<template>
    	<div class='start'>
			//先注册点击事件
			<div class='btn' @click='handelClick'>{{btnText}}</div>
		</div>
	</template>
	<script>
		export default {
		    name:'btn',
		    data () {
		        return {
		        }
		    },
		    props: {
				//声明btnText
				btnText:{
					type:String,//声明该属性为一个字符串
					default:'点击'//默认值为 点击,当外部没有传递时，将会采用这个值
				}
		    },
		    methods: {
				//这里来处理点击事件
		        handelClick(){
					//这里直接通过this.$emit来表示，通知父组件：该点击事件触发了，并且会执行父组件绑定在子组件上的on-click方法

					this.$emit('on-click');
				}
		    },
		    created () {
		    }
		}
	</script>

接下来在父组件中：
	
	//app.vue
	<template>
    	<div class='index-app'>
			//监听on-click这个方法，当子组件内的click事件被触发的时候，isClick方法便会执行
			<btn :btnText='btnText' @on-click='isClick'></btn>
		</div>
	</template>
	<script>
	//使用组件必须先引入
	import btn from './btn.vue'
		export default {
			//并且需要在这里进行声明注册这个组件,这里两步是必须的
			components:{
				btn
			},
			data(){
				return {
					btnText:'点一下'//这里来表示传入的文字
				}
			},
			methods:{
				//这里就是执行的click方法
				isClick(){
					console.log('click被执行');
				}
			}
		}
	</script>
	

这便是最简单的组件，还有更多的使用方法，请自己探索吧