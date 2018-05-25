# Vue生命周期介绍
作者：熊猫
时间: 2018年5月23日

用Vue开发已经有一些日子，但是关于Vue生命周期的问题一直没有形成一个比较明确而深刻的概念，使用的时候总是无法得心应手，于是希望写一篇文章来介绍Vue生命周期的相关知识，也算是巩固和加深自己对于Vue的理解。

## 写在介绍之前的内容
由于箭头函数的this绑定的是当前父级上下文，所以如果用箭头函数，this指向的并不是当前的Vue实例，所以没法取到数据，方法等。
以下是官方解释：
> 不要在选项属性或回调上使用箭头函数，比如 created: () => console.log(this.a) 或 vm.$watch('a', newValue => this.myMethod())。因为箭头函数是和父级上下文绑定在一起的，this 不会是如你所预期的 Vue 实例，经常导致 Uncaught TypeError: Cannot read property of undefined 或 Uncaught TypeError: this.myMethod is not a function 之类的错误。

##  beforeCreate()
此时为实例创建之前。使用这个钩子方法时,this.\$el还不存在。各种组件属性(this.\$data,methods里面的函数，computed数据等等)还没有计算出来。所以在这个钩子方法里调用数据或方法只会返回undefined。
如果是单文件组件的话，Document.title是可以访问到的。
**注意：**
如果你真的有这种需求，需要在实例未生成之前就调用相关属性或方法，那么可以使用this.\$options的属性来调用，但this.\$options是只读属性，是无法修改的。所以期望在这个钩子函数中用这个属性更改属性和方法或者组件的是不可能的。（方法是可以调用的，当然如果内部有相关引用，那么肯定会报错，因为此时还没有生成相关引用）
this.\$options.data()没有绑定上下文，要用this.\$options.data.apply(this)。这个方法只针对当前组件的data，子组件里的data就不需要绑定this了。
this.\$options.methods.functionName()可以直接调用方法;
- Vue 1.0名称: init()

- 可以使用的属性：document的相关属性，this.\$options, this.$route

- 不可使用的属性(或者说没有值的属性): this.\$el, this.\$data, 所有的data属性，所有的DOM节点，所有的方法和回调。

- 使用场景: 根据路由信息重定向，实现一个状态机等。  

##  created()
此时所有的数据已经完成了绑定，但还没有渲染在真实的DOM元素节点之上，此时this指向的所有数据和方法，包括watch/event回调都已完成。但是此时的\$el仍然没有创建，DOM也没有生成
- Vue 1.0名称: created()

- 可以使用的属性: this.$data,所有的data属性，所有的方法和回调（watch）, computed,

- 不可使用的属性(或者说没有值的属性): this.\$el,所有的DOM节点获取和操作(例:document.getElementById())

- 使用场景: ajax数据请求，data数据的初始化.

##  beforeMount()
这个时候是将虚拟的\$el渲染进文档中。此时如果获取DOM节点，是可以获取到的(如\<div\>{{ message }}\</div\>,message的具体内容在这个阶段不会填充)，但是其中的数据还不是真实的，只是变量名称。

- Vue 1.0名称: beforeCompile()

- 可以使用的属性: this.$data,所有的data属性，所有的方法和回调（watch）, computed, this.\$el(但是数据不是真实的)

- 不可使用的属性(或者说没有值的属性): DOM节点上的数据

- 使用场景: 一般来说不会在这里有业务逻辑的实现，除非你必须需要在这个钩子中拿到DOM操作。

##  mounted()
此时虚拟的el(DOM节点)被真实的vm.\$el替换，所有的数据也被渲染上去。但是这个钩子函数并不保证对应的el已在文档中，也不保证所有子组件全部挂载完成，如果需要所有子组件都挂载完成后的状态，需要使用nextTick方法。
该钩子在服务器端渲染期间不被调用。

- Vue 1.0名称: compiled()

- 可以使用的属性: this.$data,所有的data属性，所有的方法和回调（watch）, computed, this.\$el

- 不可使用的属性(或者说没有值的属性): 无

- 使用场景: 对DOM需要操作的场景

## nextTick()
这里加多一个nextTick()方法的使用。
一般有两种用法:this.$nextTick, Vue.nextTick。这里的区别只是用this绑定到当前组件实例或者用Vue绑定全局实例的方法。
理解nextTick你需要理解Vue中的DOM更新是**异步**而非同步的。当你更改某一个值时，这个改变不会即时在DOM上渲染，而是生成一个DOM更新队列，然后定一个定时器，之后渲染。一般来说，由于更新太快并不会看出有多少区别。然而，如果你在代码中需要在更改值之后操作更新后的DOM时，就没法立即操作了。因为之前更改操作并没有渲染(还在队列中)，如果你需要操作更新后的DOM，则需要使用这个方法。
(关于nextTick的详细分析，可以参考这篇文章[从Vue.js源码看nextTick机制](https://zhuanlan.zhihu.com/p/30451651))
```
export default {
    data() {
        return {
            test: 0,
        }
    },
    created() {
        this.test = 1;
        console.log('log1', this.test);
    },
    mounted() {
        this.test = 2;
        this.$nextTick(() => {
            console.log('log2', this.test);
        })
        this.test = 3;
        this.$nextTick(() => {
            console.log('log3', this.test);
        })
    }
}
// log1 1
// log2 3
// log3 3
// 页面上也只会出现最终的3，而不是2 -> 3;
```
**注意：**
nextTick也是一个异步操作，所以你的其他在nextTick之后的同步操作会在这个方法之前被实现。  

## beforeUpdate()
这里是发生在数据更新时，而DOM还没有渲染的时候。
**注意:**
这里如果进行值的操作，会触发无限循环。因为你的值在更新的同时，又会触发钩子函数（如果是不可变值，例如字符串数字等，不会重复触发，但还是不建议进行值操作）

- Vue 1.0名称: 无

- 可以使用的属性: 同上

- 不可使用的属性(或者说没有值的属性): 无

- 使用场景: 很少，能想到的是对数据的提前过滤

## updated()
这里是发生在数据更新且DOM也重新渲染后的函数。

注意事项同上

- Vue 1.0名称: 无

- 可以使用的属性: 同上

- 不可使用的属性(或者说没有值的属性): 无

- 使用场景: 数据更新后的操作DOM

## beforeDestory()
组件销毁前调用。此时整个实例仍然可用

- Vue 1.0名称: beforeDestory

- 可以使用的属性: 同上

- 不可使用的属性(或者说没有值的属性): 无

- 使用场景: 根据业务而定

## destroyed()
组件销毁后调用。此时整个实例所有数据解绑，所有监听器移除等。

- Vue 1.0名称: destoryed

- 可以使用的属性: 同上

- 不可使用的属性(或者说没有值的属性): 无

- 使用场景: 根据业务而定

## activated()
只针对keep-alive组件，组件被激活时触发。

- Vue 1.0名称: 无

- 可以使用的属性: 同上

- 不可使用的属性(或者说没有值的属性): 无

- 使用场景: 根据业务而定

## activated()
只针对keep-alive组件，组件被移除时触发。

- Vue 1.0名称: 无

- 可以使用的属性: 同上

- 不可使用的属性(或者说没有值的属性): 无

- 使用场景: 根据业务而定

本人才疏学浅，权当抛砖引玉。如果文中有错误或者各位有更详细的理解，也可以一起讨论和分享，共同进步。
另外此篇文章没有像普遍的文章带上代码和图片，如果有人觉得难以理解，可以查看下方的参考链接，有比较详细的图文代码解释。


参考文章：http://www.cnblogs.com/zhuzhenwei918/p/6903158.html
         https://segmentfault.com/a/1190000009677699
         https://github.com/vuejs/vue/issues/702
         https://segmentfault.com/q/1010000012331476
         https://blog.csdn.net/zhalcie2011/article/details/72265881
         https://stackoverflow.com/questions/47634258/what-is-nexttick-or-what-does-it-do-in-vuejs
         https://stackoverflow.com/questions/44983349/what-is-the-difference-between-updated-hook-and-watchers-in-vuejs