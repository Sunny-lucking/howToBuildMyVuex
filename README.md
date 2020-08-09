
# 手写Vuex核心原理
@[toc]
## 一、核心原理
1. Vuex本质是一个对象
2. Vuex对象有两个属性，一个是install方法，一个是Store这个类
3. install方法的作用是将store这个实例挂载到所有的组件上，注意是同一个store实例。
4. Store这个类拥有commit，dispatch这些方法，Store类里将用户传入的state包装成data，作为new Vue的参数，从而实现了state 值的响应式。
## 二、基本准备工作

我们先利用vue-cli建一个项目



![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWdrci5jbi1iai51ZmlsZW9zLmNvbS9hNDY2NDlmNS0wMTk4LTRhNmYtYWZjMC04Nzg4OGVjZTQyMTQucG5n?x-oss-process=image/format,png)



删除一些不必要的组建后项目目录暂时如下：
  

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWdrci5jbi1iai51ZmlsZW9zLmNvbS8wZmU4YzkzMi1mNjBhLTQ2NmMtYjMxNS1jZjkyZWRmNTM2YjYucG5n?x-oss-process=image/format,png)


>已经把项目放到 **github**：https://github.com/Sunny-lucking/howToBuildMyVuex  可以卑微地要个star吗。有什么不理解的或者是建议欢迎评论提出


我们主要看下App.vue,main.js,store/index.js

代码如下:
  
App.vue
```
<template>
  <div id="app">
    123
  </div>
</template>
```

store/index.js

```js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

export default new Vuex.Store({
  state: {
  },
  mutations: {
  },
  actions: {
  },
  modules: {
  }
})

```
main.js

```js
import Vue from 'vue'
import App from './App.vue'
import store from './store'

Vue.config.productionTip = false

new Vue({
  store,
  render: h => h(App)
}).$mount('#app')
```


现在我们启动一下项目。看看项目初始化有没有成功。


![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWdrci5jbi1iai51ZmlsZW9zLmNvbS85N2QzZGE2Yy0zNTVlLTRhYjAtYTBiMi00NThkNzIwM2QyYjUucG5n?x-oss-process=image/format,png)


ok，没毛病，初始化成功。

现在我们决定创建自己的Vuex，于是创建myVuex.js文件

目前目录如下


![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWdrci5jbi1iai51ZmlsZW9zLmNvbS9jNWQ5YWRkNy0yNDMyLTRhNzUtYTNhOC0wNDE4M2UzMWU3ODcucG5n?x-oss-process=image/format,png)


再将Vuex引入 改成我们的myVuex

```js
//store/index.js
import Vue from 'vue'
import Vuex from './myVuex' //修改代码

Vue.use(Vuex)

export default new Vuex.Store({
  state: {
  },
  mutations: {
  },
  actions: {
  },
  modules: {
  }
})
```

## 三、剖析Vuex本质
  

先抛出个问题，Vue项目中是怎么引入Vuex。

1. 安装Vuex，再通过`import Vuex from 'vuex'`引入
2. 先 var store = new Vuex.Store({...}),再把store作为参数的一个属性值，new Vue({store})
3. 通过Vue.use(Vuex) 使得每个组件都可以拥有store实例

从这个引入过程我们可以发现什么？
1. 我们是通过new Vuex.store({})获得一个store实例，也就是说，我们引入的Vuex中有Store这个类作为Vuex对象的一个属性。因为通过import引入的，**实质上就是一个导出一个对象的引用**。

所以我们可以初步假设

```js
Class Store{
  
}

let Vuex = {
  Store
}
```
2. 我们还使用了Vue.use(),而Vue.use的一个原则就是执行对象的install这个方法

所以，我们可以再一步 假设Vuex有有install这个方法。

```js
Class Store{
  
}
let install = function(){
  
}

let Vuex = {
  Store,
  install
}
```

到这里，你能大概地将Vuex写出来吗？

很简单，就是将上面的Vuex对象导出，如下就是myVuex.js

```js
//myVuex.js
class Store{

}
let install = function(){

}

let Vuex = {
    Store,
    install
}

export default Vuex
```

我们执行下项目，如果没报错，说明我们的假设没毛病。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWdrci5jbi1iai51ZmlsZW9zLmNvbS9iNTdjOTZiYS00MzRhLTQ4YzAtYjRkNi0zMWJiMzc3NzNjNjcucG5n?x-oss-process=image/format,png)

天啊，没报错。没毛病！

## 四、分析Vue.use
Vue.use(plugin);

（1）参数


```
{ Object | Function } plugin
```


（2）用法

安装Vue.js插件。如果插件是一个对象，必须提供install方法。如果插件是一个函数，它会被作为install方法。调用install方法时，会将Vue作为参数传入。install方法被同一个插件多次调用时，插件也只会被安装一次。

关于如何上开发Vue插件，请看这篇文章，非常简单，不用两分钟就看完：[如何开发 Vue 插件？](https://mp.weixin.qq.com/s?__biz=MzU5NDM5MDg1Mw==&mid=2247483874&idx=1&sn=ac6c9cf2629068dec3e5da8aa3e29364&chksm=fe00bbc8c97732dea7be43e903a794229876d8ab6c9381f2388ba22886fba7776b7b34b7af86&token=1885963052&lang=zh_CN#rd)

（3）作用

注册插件，此时只需要调用install方法并将Vue作为参数传入即可。但在细节上有两部分逻辑要处理：

1、插件的类型，可以是install方法，也可以是一个包含install方法的对象。

2、插件只能被安装一次，保证插件列表中不能有重复的插件。

（4）实现



```js
Vue.use = function(plugin){
	const installedPlugins = (this._installedPlugins || (this._installedPlugins = []));
	if(installedPlugins.indexOf(plugin)>-1){
		return this;
	}
	<!-- 其他参数 -->
	const args = toArray(arguments,1);
	args.unshift(this);
	if(typeof plugin.install === 'function'){
		plugin.install.apply(plugin,args);
	}else if(typeof plugin === 'function'){
		plugin.apply(null,plugin,args);
	}
	installedPlugins.push(plugin);
	return this;
}
```
1、在Vue.js上新增了use方法，并接收一个参数plugin。

2、首先判断插件是不是已经别注册过，如果被注册过，则直接终止方法执行，此时只需要使用indexOf方法即可。

3、toArray方法我们在就是将类数组转成真正的数组。使用toArray方法得到arguments。除了第一个参数之外，剩余的所有参数将得到的列表赋值给args，然后将Vue添加到args列表的最前面。这样做的目的是保证install方法被执行时第一个参数是Vue，其余参数是注册插件时传入的参数。

4、由于plugin参数支持对象和函数类型，所以通过判断plugin.install和plugin哪个是函数，即可知用户使用哪种方式祖册的插件，然后执行用户编写的插件并将args作为参数传入。

5、最后，将插件添加到installedPlugins中，保证相同的插件不会反复被注册。(~~**让我想起了曾经面试官问我为什么插件不会被重新加载！！！哭唧唧，现在总算明白了**)

## 五、完善install方法
我们前面提到 **通过Vue.use(Vuex) 使得每个组件都可以拥有store实例**。

这是什么意思呢？？？

来看mian.js


```js
import Vue from 'vue'
import App from './App.vue'
import store from './store'

Vue.config.productionTip = false;

new Vue({
  store,
  render: h => h(App)
}).$mount('#app');
```

我们可以发现这里只是将store ，也就是store/index.js导出的store实例，作为Vue 参数的一部分。

但是这里就是有一个问题咯，这里的Vue 是根组件啊。也就是说目前只有根组件有这个store值，而其他组件是还没有的，所以我们需要让其他组件也拥有这个store。

因此，install方法我们可以这样完善


```
let install = function(Vue){
    Vue.mixin({
        beforeCreate(){
            if (this.$options && this.$options.store){ // 如果是根组件
                this.$store = this.$options.store
            }else { //如果是子组件
                this.$store = this.$parent && this.$parent.$store
            }
        }
    })
}
```

解释下代码：
1. 参数Vue，我们在第四小节分析Vue.use的时候，再执行install的时候，将Vue作为参数传进去。
2. mixin的作用是将mixin的内容混合到Vue的初始参数options中。相信使用vue的同学应该使用过mixin了。
3. 为什么是beforeCreate而不是created呢？因为如果是在created操作的话，`$options`已经初始化好了。
4. 如果判断当前组件是根组件的话，就将我们传入的store挂在到根组件实例上，属性名为`$store`。
5. 如果判断当前组件是子组件的话，就将我们根组件的`$store`也复制给子组件。注意是**引用的复制**，因此每个组件都拥有了同一个`$store`挂载在它身上。

这里有个问题，为什么判断当前组件是子组件，就可以直接从父组件拿到`$store`呢？这让我想起了曾经一个面试官问我的问题：**父组件和子组件的执行顺序**？

>A：父beforeCreate-> 父created -> 父beforeMounte  -> 子beforeCreate ->子create ->子beforeMount ->子 mounted -> 父mounted

可以得到，在执行子组件的beforeCreate的时候，父组件已经执行完beforeCreate了，那理所当然父组件已经有`$store`了。

## 六、实现Vuex的state

```
    <p>{{this.$store.state.num}}</p>
```
我们都知道，可以通过这个 语句获得 state的值
但是我们在Store类里还没实现，显然，现在就这样取得话肯定报错。

前面讲过，我们是这样使用Store的

```js
export default new Vuex.Store({
  state: {
    num:0
  },
  mutations: {
  },
  actions: {
  },
  modules: {
  }
})
```
也就是说，我们把这个对象
```
{
  state: {
    num:0
  },
  mutations: {
  },
  actions: {
  },
  modules: {
  }
}
```
当作参数了。

那我们可以直接在Class Store里，获取这个对象


```js
class Store{
    constructor(options){
        this.state = options.state || {}
        
    }
}
```
那这样是不是可以直接使用了呢？

试一下呗！

```js
//App.vue
<template>
  <div id="app">
    123
    <p>{{this.$store.state.num}}</p>
  </div>
</template>
```
运行结果：

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWdrci5jbi1iai51ZmlsZW9zLmNvbS8xMmM4Njc4MC03YWYyLTRkZTctODczZi02ZTQxMDk0NWVjYTEucG5n?x-oss-process=image/format,png)

太赞了吧，怎么会这么简单。。。不敢相信。

哦不，当然没有这么简单，我们忽略了一点，state里的值也是响应式的哦，我们这样可没有实现响应式。

>曾经面试官问我Vuex和全局变量比有什么区别。这一点就是注意区别吧

那要怎么实现响应式呢？ 我们知道，我们new Vue（）的时候，传入的data是响应式的，那我们是不是可以new 一个Vue，然后把state当作data传入呢？ 没有错，就是这样。


```js
class Store{

    constructor(options) {
        this.vm = new Vue({
            data:{
                state:options.state
            }
        })
    }

}
```
现在是实现响应式了，但是我们怎么获得state呢？好像只能通过`this.$store.vm.state`了？但是跟我们平时用的时候不一样，所以，是需要转化下的。

我们可以给Store类添加一个state属性。这个属性自动触发get接口。


```js
class Store{

    constructor(options) {
        this.vm = new Vue({
            data:{
                state:options.state
            }
        })

    }
    //新增代码
    get state(){
        return this.vm.state
    }


}
```

这是ES6，的语法，有点类似于Object.defineProperty的get接口


![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWdrci5jbi1iai51ZmlsZW9zLmNvbS8zZjExMWQ5My1hNjM5LTRjNTUtODNhZS0yMGFlMTM4ZmU5ODYucG5n?x-oss-process=image/format,png)
成功实现。

## 七、实现getter


```js
//myVuex.js
class Store{

    constructor(options) {
        this.vm = new Vue({
            data:{
                state:options.state
            }
        })
        // 新增代码
        let getters = options.getter || {}
        this.getters = {}
        Object.keys(getters).forEach(getterName=>{
            Object.defineProperty(this.getters,getterName,{
                get:()=>{
                    return getters[getterName](this.state)
                }
            })
        })

    }
    get state(){
        return this.vm.state
    }
}

```
我们把用户传进来的getter保存到getters数组里。

最有意思的是经常会有面试题问：**为什么用getter的时候不用写括号**。要不是我学到这个手写Vuex，也不会想不明白，原来这个问题就像问我们平时写个变量，为什么不用括号一样。（如`{{num}}`,而不是`{{num()}}`）

原来就是利用了Object.defineProperty的get接口。

ok，现在来试一下，getter可不可以使用。


```js
//store/index.js
import Vue from 'vue'
import Vuex from './myVuex'

Vue.use(Vuex)

export default new Vuex.Store({
  state: {
    num:0
  },
  // 新增测试代码
  getter:{
    getNum:(state)=>{
      return state.num
    }
  },
  mutations: {
  },
  actions: {
  },
})
```


```js
<template>
  <div id="app">
    123
    <p>state：{{this.$store.state.num}}</p>
    <p>getter:{{this.$store.getters.getNum}}</p>
  </div>
</template>
```


![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWdrci5jbi1iai51ZmlsZW9zLmNvbS8zNmM2MTVkYS1iNWVhLTQ2NDItOTcwNi1mYTcwMTA5MmM5MzAucG5n?x-oss-process=image/format,png)

完美。毫无事故。
## 八、实现mutation


```js
//myVuex.js
class Store{

    constructor(options) {
        this.vm = new Vue({
            data:{
                state:options.state
            }
        })

        let getters = options.getter || {}
        this.getters = {}
        Object.keys(getters).forEach(getterName=>{
            Object.defineProperty(this.getters,getterName,{
                get:()=>{
                    return getters[getterName](this.state)
                }
            })
        })
        //新增代码
        let mutations = options.mutations || {}
        this.mutations = {}
        Object.keys(mutations).forEach(mutationName=>{
            this.mutations[mutationName] = (arg)=> {
                mutations[mutationName](this.state,arg)
            }
        })

    }
    get state(){
        return this.vm.state
    }
}
```


mutations跟getter一样，还是用mutations对象将用户传入的mutations存储起来。

但是怎么触发呢？回忆一下，我们是怎么触发mutations的。

```
this.$store.commit('incre',1)
```

对，是这种形式的。可以看出store对象有commit这个方法。而commit方法触发了mutations对象中的某个对应的方法，因此我们可以给Store类添加commit方法

```js
//myVuex.js
class Store{
    constructor(options) {
        this.vm = new Vue({
            data:{
                state:options.state
            }
        })

        let getters = options.getter || {}
        this.getters = {}
        Object.keys(getters).forEach(getterName=>{
            Object.defineProperty(this.getters,getterName,{
                get:()=>{
                    return getters[getterName](this.state)
                }
            })
        })
        
        let mutations = options.mutations || {}
        this.mutations = {}
        Object.keys(mutations).forEach(mutationName=>{
            this.mutations[mutationName] =  (arg)=> {
                mutations[mutationName](this.state,arg)
            }
        })

    }
    //新增代码
    commit(method,arg){
        this.mutations[method](arg)
    }
    get state(){
        return this.vm.state
    }
}
```

好了，现在来测试一下。


```js
<template>
  <div id="app">
    123
    <p>state:{{this.$store.state.num}}</p>
    <p>getter:{{this.$store.getters.getNum}}</p>
    <button @click="add">+1</button>
  </div>
</template>
//新增测试代码
<script>
  export default {
      methods:{
          add(){
              this.$store.commit('incre',1)
          }
      }
  }
</script>
```
store/index.js
```js
//store/index.js
import Vue from 'vue'
import Vuex from './myVuex'

Vue.use(Vuex)

export default new Vuex.Store({
  state: {
    num:0
  },
  getter:{
    getNum:(state)=>{
      return state.num
    }
  },
  // 新增测试代码
  mutations: {
    incre(state,arg){
        state.num += arg
    }
  },
  actions: {
  },
})
```

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWdrci5jbi1iai51ZmlsZW9zLmNvbS84NmU0MTk2Ni1lZTM2LTRjNzQtYTc3NS03MjQ4ODc0MmIyMmUucG5n?x-oss-process=image/format,png)

运行成功。

## 九、实现actions

当会实现mutations后，那actions的实现也很简单，很类似，不信看代码：

```js
//myVuex.js
class Store{
    constructor(options) {
        this.vm = new Vue({
            data:{
                state:options.state
            }
        })

        let getters = options.getter || {}
        this.getters = {}
        Object.keys(getters).forEach(getterName=>{
            Object.defineProperty(this.getters,getterName,{
                get:()=>{
                    return getters[getterName](this.state)
                }
            })
        })

        let mutations = options.mutations || {}
        this.mutations = {}
        Object.keys(mutations).forEach(mutationName=>{
            this.mutations[mutationName] =  (arg)=> {
                mutations[mutationName](this.state,arg)
            }
        })
        //新增代码
        let actions = options.actions
        this.actions = {}
        Object.keys(actions).forEach(actionName=>{
            this.actions[actionName] = (arg)=>{
                actions[actionName](this,arg)
            }
        })

    }
    // 新增代码
    dispatch(method,arg){
        this.actions[method](arg)
    }
    commit(method,arg){
        console.log(this);
        this.mutations[method](arg)
    }
    get state(){
        return this.vm.state
    }
}
```
一毛一样，不过有一点需要解释下，就是这里为什么是传this进去。这个this代表的就是store实例本身
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWdrci5jbi1iai51ZmlsZW9zLmNvbS8xYjY4N2NhNS03NThiLTQ3N2ItOTViOC1kMGZiZmQ2MTFiZDgucG5n?x-oss-process=image/format,png)
这是因为我们使用actions是这样使用的：

```js
  actions: {
    asyncIncre({commit},arg){
        setTimeout(()=>{
          commit('incre',arg)
        },1000)
    }
  },
```

其实{commit} 就是对this，即store实例的解构
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWdrci5jbi1iai51ZmlsZW9zLmNvbS8yMDc4Y2E1Zi1hMWU0LTQ4NjQtOWY1OS05ZTc0Y2RjMjQ1ODYucG5n?x-oss-process=image/format,png)
。

那我们来测试一下。

```js

<template>
  <div id="app">
    123
    <p>state:{{this.$store.state.num}}</p>
    <p>getter:{{this.$store.getters.getNum}}</p>
    <button @click="add">+1</button>
    <button @click="asyncAdd">异步+2</button>
  </div>
</template>

<script>
  export default {
      methods:{
          add(){
              this.$store.commit('incre',1)
          },
          asyncAdd(){
              this.$store.dispatch('asyncIncre',2)
          }
      }
  }
</script>
```

store/index.js
```js
//store/index.js
import Vue from 'vue'
import Vuex from './myVuex'

Vue.use(Vuex)

export default new Vuex.Store({
  state: {
    num:0
  },
  getter:{
    getNum:(state)=>{
      return state.num
    }
  },
  mutations: {
    incre(state,arg){
        state.num += arg
    }
  },
  //新增测试代码
  actions: {
    asyncIncre({commit},arg){
        setTimeout(()=>{
          commit('incre',arg)
        },1000)
    }
  },
})
```

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWdrci5jbi1iai51ZmlsZW9zLmNvbS83YWJkOTBhNS1iM2YyLTRiOTMtODYyNC1lYjEyOTI5NmEwYmIucG5n?x-oss-process=image/format,png)

oh my god，居然出错了，它这里错误 说的是执行到这里发现这里的this为undefined。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWdrci5jbi1iai51ZmlsZW9zLmNvbS85OWQ4YmI1YS01NGExLTQyY2YtOWI4OC1hMWZlOWM5YTY1NTYucG5n?x-oss-process=image/format,png)

不过，不对啊，我们在实现mutation的时候也执行到这里了啊，而且执行成功了啊。

来分析一下：


```js
this.$store.commit('incre',1)
```
执行这段代码的时候，执行commit的时候，this是谁调用就指向谁，所以this指向`$store`。


```js
this.$store.dispatch('asyncIncre',2)
```
执行这段代码，就会执行


```js
asyncIncre({commit},arg){
    setTimeout(()=>{
      commit('incre',arg)
    },1000)
}
```
发现问题了吧？？ 谁调用commit？？是`$store`吗？并不是。所以要解决这个问题，我们必须换成箭头函数


```js
//myVuex.js
class Store{
    constructor(options) {
        this.vm = new Vue({
            data:{
                state:options.state
            }
        })

        let getters = options.getter || {}
        this.getters = {}
        Object.keys(getters).forEach(getterName=>{
            Object.defineProperty(this.getters,getterName,{
                get:()=>{
                    return getters[getterName](this.state)
                }
            })
        })

        let mutations = options.mutations || {}
        this.mutations = {}
        Object.keys(mutations).forEach(mutationName=>{
            this.mutations[mutationName] =  (arg)=> {
                mutations[mutationName](this.state,arg)
            }
        })

        let actions = options.actions
        this.actions = {}
        Object.keys(actions).forEach(actionName=>{
            this.actions[actionName] = (arg)=>{
                actions[actionName](this,arg)
            }
        })

    }
    dispatch(method,arg){
        this.actions[method](arg)
    }
    // 修改代码
    commit=(method,arg)=>{
        console.log(method);
        console.log(this.mutations);
        this.mutations[method](arg)
    }
    get state(){
        return this.vm.state
    }
}
```

再来测试


![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWdrci5jbi1iai51ZmlsZW9zLmNvbS8wZjhlZjM2YS1jNjc3LTRhZDItODlkNy04MDc5OGQ3Nzk3MjEucG5n?x-oss-process=image/format,png)

完美收官！！！！

>补充:有群友问到一个问题，我觉得很有意思，就是说直接通过`$store.state.xx = ""`。可以吗？其实这样赋值也不会有问题，而且state依旧是响应式的。那么为什么用commit来多此一举呢？    

**vuex能够记录每一次state的变化记录，保存状态快照，实现时间漫游／回滚之类的操作**。




我有想到一件有意思的事情，要是说我们要实现一个最简单的Vuex，其实只实现state不就好了，其他的getter啊，action，commit都不实现。有种轻装上阵的感觉。其实也能实现。

而这样实现后发现其实跟全局变量差不多，只不过state是响应式的。

有什么不理解的或者是建议欢迎评论提出

感谢您也恭喜您看到这里，我可以卑微的求个star吗！！！
>github：https://github.com/Sunny-lucking/howToBuildMyWebpack
