# vue 生命周期深入

TANGiMING  2018-10-26

##一. 生命周期钩子函数

beforeCreate：实例初始化之后，this指向创建的实例，不能访问到data、computed、watch、methods上的方法和数据，常用于初始化非响应式变量；

created：实例创建完成，可访问data、computed、watch、methods上的方法和数据，未挂载到DOM，不能访问到$el属性，$ref属性内容为空数组，常用于简单的ajax请求，页面的初始化

beforeMount：在挂载开始之前被调用，beforeMount之前，会找到对应的template，并编译成render函数


mounted：实例挂载到DOM上，此时可以通过DOM API获取到DOM节点，$ref属性可以访问，常用于获取VNode信息和操作，ajax请求

beforeupdate：响应式数据更新时调用，发生在虚拟DOM打补丁之前，适合在更新之前访问现有的DOM，比如手动移除已添加的事件监听器

updated：虚拟 DOM 重新渲染和打补丁之后调用，组件DOM已经更新，可执行依赖于DOM的操作，避免在这个钩子函数中操作数据，可能陷入死循环

beforeDestroy：实例销毁之前调用。这一步，实例仍然完全可用，this仍能获取到实例，常用于销毁定时器、解绑全局事件、销毁插件对象等操作；

destroyed：实例销毁后调用，调用后，Vue 实例指示的所有东西都会解绑定，所有的事件监听器会被移除，所有的子实例也会被销毁



### 注意：

created阶段的ajax请求与mounted请求的区别：前者页面视图未出现，如果请求信息过多，页面会长时间处于白屏状态
mounted 不会承诺所有的子组件也都一起被挂载。如果你希望等到整个视图都渲染
完毕，可以用 vm.$nextTick

vue2.0之后主动调用$destroy()不会移除dom节点，作者不推荐直接destroy这种做法，如果实在需要这样用可以在这个生命周期钩子中手动移除dom节点

##二. 单个组件的生命周期，现根据实际代码执行情况分析：

	<template>
	    <div>
	        <h3>单组件</h3>
	        <el-button @click="dataVar += 1">更新 {{dataVar}}</el-button>
	        <el-button @click="handleDestroy">销毁</el-button>
	    </div>
	</template>
----
	export default {
    data() {
        return {
            dataVar: 1
        }
    },
    beforeCreate() {
        this.compName = 'single'
        console.log(`--${this.compName}--beforeCreate`)
    },
    created() {
        console.log(`--${this.compName}--created`)
    },
    beforeMount() {
        console.log(`--${this.compName}--beforeMount`)
    },
    mounted() {
        console.log(`--${this.compName}--mounted`)
    },
    beforeUpdate() {
        console.log(`--${this.compName}--beforeUpdate`)
    },
    updated() {
        console.log(`--${this.compName}--updated`)
    },
    beforeDestroy() {
        console.log(`--${this.compName}--beforeDestroy`)
    },
    destroyed() {
        console.log(`--${this.compName}--destroyed`)
    },
    methods: {
        handleDestroy() {
            this.$destroy()
	        }
	    }
	}
	
	beforeCreate
	created
	beforeMount
	mounted
当data变化时

	beforeUpdate
	updated
	
当组件销毁时

	beforeDestroy
	destroyed
	
从打印结果可以看出:

初始化组件时，仅执行了beforeCreate/Created/beforeMount/mounted四个钩子函数

当改变data中定义的变量（响应式变量）时，会执行beforeUpdate/updated钩子函数

当切换组件（当前组件未缓存）时，会执行beforeDestory/destroyed钩子函数

初始化和销毁时的生命钩子函数均只会执行一次，beforeUpdate/updated可多次执行

##三. 父子组件的生命周期
将单组件作为基础组件（由于props在beforeCreate()中未初始化），需要做如下更改：

	props: {
    compName: {
        type: String,
        default: 'single'
    }
	},
	beforeCreate() {
	    // this.compName = 'single'
	    // console.log(`--${this.compName}--beforeCreate`)
	
	    console.log(` --data未初始化--beforeCreate`)
	},

父组件代码如下：

	<template>
    <div class="complex">
        <h3>复杂组件</h3>
        <lifecycle-single compName="child"></lifecycle-single>
    </div>
	</template>
	
-----

	const COMPONENT_NAME = 'complex'

	import LifecycleSingle from './LifeCycleSingle'
	
	export default {
	    beforeCreate() {
	        console.log(`--${COMPONENT_NAME}--beforeCreate`)
	    },
	    created() {
	        console.log(`--${COMPONENT_NAME}--created`)
	    },
	    beforeMount() {
	        console.log(`--${COMPONENT_NAME}--beforeMount`)
	    },
	    mounted() {
	        console.log(`--${COMPONENT_NAME}--mounted`)
	    },
	    beforeUpdate() {
	        console.log(`--${COMPONENT_NAME}--beforeUpdate`)
	    },
	    updated() {
	        console.log(`--${COMPONENT_NAME}--updated`)
	    },
	    beforeDestroy() {
	        console.log(`--${COMPONENT_NAME}--beforeDestroy`)
	    },
	    destroyed() {
	        console.log(`--${COMPONENT_NAME}--destroyed`)
	    },
	    components: {
	        LifecycleSingle
	    }
	}
	
	当初始化时，打印：
		--complex--beforeCreate
		--complex--created
		--complex--beforeMount
			--data未初始化--beforeCreate
			--child--created
			--child--beforeMount
			--child--mounted
		--complex--mounted
		--complex--beforeUpdate
		--complex--updated
		
	当子组件data发生变化时，打印：
	
		--child--beforeUpdate
		--child--updated
	
	当父组件data发生变化时：
	
		--complex--beforeUpdate
		--complex--updated
	
	当props发生变化时：
	
		--complex--beforeUpdate
		--2--beforeUpdate
		--2--updated
		--complex--updated
	
	当子组件销毁时：
	
		--child--beforeDestroy
		--child--destroyed
	
	当父组件销毁时：
	
		--complex--beforeDestroy
			--child--beforeDestroy
		--complex--destroyed
			--child--destroyed
			
###从打印结果可以看出:

仅当子组件完成挂载后，父组件才会挂载
当子组件完成挂载后，父组件会主动执行一次beforeUpdate/updated钩子函数（仅首次）
父子组件在data变化中是分别监控的，但是在更新props中的数据是关联的（可实践）
销毁父组件时，先将子组件销毁后才会销毁父组件


	
	
	







