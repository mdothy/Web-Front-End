#### Vue 组件
###### 组件开发使得vue变得灵活，同时增加了html的扩展性，所有的组件都是Vue实例，因此，我们可以在上面定义它的各种属性和方法，通过在不同组件之间传递属性和方法达到控制。


##### 组件初识
```
<div id="app">
    <helloworld></helloworld>
</div>
Vue.component('helloworld', {
    template: '<li>{{msg}}</li>',
    data() {
        return {
            msg: "hello"
        }
    }
})

new Vue({
    el: '#app'
})
    这是通过定义组件名将组件注册到Vue上，再为它定义数据和DOM属性来实现
```
```    
<div id="app"></div>    
const helloworld = {
    template: '<h3>{{}}<h3>',
    data() {
        return {: "datta"
        }
    }
}
Vue.component('helloworld', component)

new Vue({
    el: '#app',
    template: '<helloworld></helloworld>'
})
        
      这是同一个例子的不同写法，只不过自定义的属性名需要实例中进行定义
```

##### 组件通信：vue的组件通信主要有父子组件和非父子组件，这里我们就侧重说一个父子组件通信。组件通信的目低主要是为了实现数据传递和数据共享
```
父子组件
 <div id="app">当前请输入:{{text}}
    <com-input v-on:change="handleChange" :text="text"></com-input>
</div>  
Vue.component('com-input', {
    props: {
        text: {
            type: String,
        default:
            "请输入内容"
        }
    },
    template: '<input v-model="msg" v-on:change="handleChange"/>',
    data() {
        return {
            msg: this.text
        }
    },
    methods: {
        handleChange(e) {
            this.$emit('change', this.msg)
        }
    }
})

new Vue({
    el: '#app',
    data() {
        return {
            text: "hello world!"
        }
    },
    methods: {
        handleChange(val) {
            this.text = val
        }
    }
})		

我们在input和msg中通过V-model做了一个数据的双向绑定，在注册的com-input中定义了ext属性和handleChange方法，当input中发生变化时，触发change事件，这个事件会传递到Vue实例的handleChange方法中，导致text属性发生变化，从而影响到最终的text文本呈现。
```

```
动态组件
<div id="app">
    <button v-on:click="changeType">changeComponent</button>
    <component v-bind:is="currentComp"></component>
</div>   
new Vue({
    el: '#app',
    data() {
        return {
            type: 'a'
        }
    },
    computed: {
        currentComp() {
            return this.type === 'a' ? 'com-a': 'com-b'
        }
    },
    components: {
        'com-a': {
            template: '<h3>first component</h3>'
        },
        'com-b': {
            template: '<h3>second component</h3>'
        }
    },
    methods: {
        changeType() {
            this.type = this.type === 'a' ? 'b': 'a';
        }
    }
})     
        
        这个就比较有意思了，通过button来触发事件，控制type属性值，is属性的改变用于切换组件。
```
##### 总结：
###### Vue的组件具有很多特性，在这里就先介绍一些，在开发过程中，需要不断的尝试，弄清组件通信的机制，才能更灵活的使用Vue进行开发，使组件间的通信逻辑在达到目的的情况下更加简洁




