

# Vue 基础学习

#### 引用

下载 vue.js --- 引入

和jQuery一样

#### 使用

##### v-once

~~~js
<div id="app" v-once>
    {{ message }}
</div>

var app = new Vue({
    el: '#app',
    data: {
        message: 'Hello Vue Js!'
    }
})
~~~

##### v-bind

~~~js
<div id="app-2">
	<p v-bind:title="message">鼠标悬停</p>
</div>

var app2 = new Vue({
    el: '#app-2',
    data: {
        message: '此页面创建于' + new Date().toLocaleDateString(),
    }
})

~~~

##### v-if

~~~js
<div id="app-3" v-if="errorMessage">{{ errorMessage }}</div>

var app3 = new Vue({
    el: '#app3',
    data: {
        errorMessage: '用户名或密码错误',
    }
})

~~~

##### v-for

~~~js
<div id="app-4">
    <ul >
    	<li v-for="topic in topics">{{ topic.title }}---{{ topic.body }}</li>
	</ul>
</div>

var app4 = new Vue({
    el: '#app-4',
    data: {
        topics: [
            {
                id: 1,
                title: '测试1',
                body: 'larabbs 测试内容2'
            },
            {
                id: 2,
                title: '测试2',
                body: 'larabbs 测试内容2'
            }, 
    		{
                id: 3,
                title: '测试3',
                body: 'larabbs 测试内容3'
			}
        ]
    }
})
~~~

##### v-on

##### v-model

~~~js
 <div id="app-5">
        <h1 v-if="love">I Love You To!</h1>
        <p>{{ message }}</p>
        <input v-model="message" v-on:keypress="isLove" />
</div>

var app5 = new Vue({
    el: '#app-5',
    data: {
        message: 'Hello Vue Js!',
        love: false,
    },
    methods: {
        isLove: function(){
            if(this.message == "I love"){
                this.love = true;
            }else{
                this.love = false;
            }
        }
    }
})
~~~



### 组件系统

使用小型，独立通常可复用的组件构建大型应用

注册组件 --- 在 Vue 中就是初始化的一个 Vue 实例

~~~js
<div class="app-7">
    <ol>
    	<todo-item></todo-item>
    </ol>
</div>


// 这样调用的 todo-item 组件 都是一样的
Vue.comonent('todo-item', {
	template: '<li>这是代办事项</li>'
})

~~~

组件中的文字 设定为变量

~~~js
Vue.component('todo-item', {
	props: ['todo'],
	template: '<li>{{ todo.text }}</li>'
})
~~~

传入变量

~~~js
var app7 = new Vue({
    el: '.app-7',
    data: {
        groceryList: [
            {
                id: 0,
                text: '蔬菜',
            },
            {
                id: 1,
                text: '水果',
            },
            {
                id: 2,
                text: '牛奶',
            },
        ]
    }
})
~~~

html 调用

~~~vue
<div class="app-7">
    <ol>
        <!-- todo 为 vue 组件中使用 -->
    	<todo-item
    		v-for="item in groceryList"
    		v-bind:todo="item"
    		v-bind:id="item.id"
        ></todo-item>
    </ol>
</div>
~~~

#### 组件的应用模板

~~~html
<div id="app">
    <!-- 导航栏 -->
    <app-nav></app-nav>
    <!-- 主内容 -->
    <app-view>
        <!-- 侧边栏 -->
        <app-sidebar></app-sidebar>
        <!-- 主要内容 -->
        <app-content></app-content>
    </app-view>
    <!-- 底部 -->
    <app-footer></app-footer>
</div>
~~~



##### Vue 生命周期

~~~js
<div class="app-8">
    <p>{{ message }}</p>
</div>

var app8 = new Vue({
    el: '.app-8',
    data: {
        message: 'hi Vue js!',
    },

    beforeCreate: function(){
        console.log('beforeCreated');

    },

    mounted: function(){
        console.log('mounted');
    },
    beforUpdate: function(){
        console.log('beforeUpdate');
    },
    updated: function(){
        console.log('beforeUpdate');
    }

})

setTimeout(function(){
    app8.message = "change ......";
}, 3000)
~~~

#### 插值

1. {{ msg }}

2. html

   用到 `v-html`

   ~~~html
   <p v-html="rawHtml"></p>
   ~~~

3. 属性[attribute]

   用到 `v-bind`

   ~~~html
   <div v-bind:id="dynamicId"></div>
   ~~~

   

#### `v-bind`, `v-on` 缩写

#### 动态参数

`v-bind` 缩写

 `v-bind:` 删除了，只有一个`:`

~~~vue
完整写法
<a v-bind:href="url">百度一下啊</a>
缩写
<a :href="url">百度一下啊</a>

动态参数写法
<a v-bind:[attribute]="url">百度一下啊</a>
缩写
<a :[attribute]="url">百度一下啊</a>
~~~

`v-on` 缩写

`v-on:`改为 `@` 了

~~~vue
完整写法
<a v-on:click="dosomething">百度一下啊</a>
缩写
<a @click="dosomething">百度一下啊</a>

动态参数
<a v-on:[enentName]="dosomething">百度一下啊</a>
缩写
<a @[enentName]="dosomething">百度一下啊</a>
~~~

