## vue初学——简单TodoList实现

### 〇、引入vue.js文件

`` <script type="text/javascript" src="./vue.js"></script>``

### 一、创建一个新vue示例

```javascript
var app = new Vue({
    el: '#app', //要绑定的元素id名字
    components: {}, //绑定的组件
    data: {}, //数据
    methods: {} //方法
})
```

### 二、不操作dom来显示数据

+ 在app中数据项写 `` list: []``，添加数据；
+ 仅在html中写，``<li v-for="for item in list">{{ item }}</li>``，即遍历list中所有元素并显示在li中，其实就相当于之前js里的for in；

### 三、点击提交动态生成列表

##### 这里要用到绑定事件，回忆一下之前的绑定事件方法：

```javascript
//原生js的绑定与解绑：
var node = document.querySelector('选择器');
node.addEventLitsener('事件类型','回调函数','是否冒泡');
node.removeEventLitsener('回调函数');
//jQuery绑定与解绑：
var $node = $('选择器');
node.on('事件类型','回调函数');
node.off('回调函数');
```

##### vue中的绑定事件

　　**可以用 `v-on` 指令监听 DOM 事件，并在触发时运行一些 JavaScript 代码。**

+ 在要绑定的元素中：``<button v-on:click='handleBtnClick'>提交</button>``。也可以省写为：``@click='handleBtnClick'``；
+ 在vue的methods中添加函数(方法)：``methods: {
  				handleBtnClick: function() {}
  			}``；

　　**`v-model` 指令，它能轻松实现表单输入和应用状态之间的双向绑定**，也就是说，在vue中修改立马会在页面中显示，反向也一样。

+ `` <input type="text" v-model='inputValue'>``，即将input得到的数据用inputValue表示，修改一个 另一个立马会跟着变，这是vue帮我们实现好了的——响应式。
+ 在vue对象里操作数据 要用``this.xx``，这里就是``this.list.push(this.inputValue);``

### 四、前端模块组件化

　　**在 Vue 里，一个组件本质上是一个拥有预定义选项的一个 Vue 实例。**

　　这里将这里的列表单独变成一个component（组件）：

#### 1、全局组件

+ 直接用Vue.component('组件名(驼峰命名法)', '组件对象')
+ 组件对象里这里用到了这几种属性：props(用于元件传递参数), templates(形成组件的模板,以后都按照这个来生成), methods(组件里的方法);
+ 假如组件名为 TodoItem, 则html中标签名(组件实例化)为todo-item(要有短横线);
+ ``v-bind:content='item'``即组件里有content这个属性, 它的值由item获得

```javascript
Vue.component('TodoItem',{
    props: ['content'],
    template: "<li>{{ content }}</li>"
})
<todo-item v-for='item in list'
			v-bind:content='item'>
</todo-item>
```

#### 2、局部组件

　　其实和上面差不多，只是定义的时候变为变量赋值的形式，且要在vue对象中进行注册, 即多声明一个属性 ``components: {TodoItem:TodoItem}``

### 五、点击条目删除

　　**这里是利用组件来删除条目, 所以是要将数据从子组件传到父组件，这里用事件来实现．**

+ 在组件的template中加上事件绑定, 变为``template: "<li @click='handleItemClick'>{{ content }}</li>"``, 即每次点击li就调用handleItemClick回调函数;
+ 其中handleItemClick回调函数中又调用vue中的 $emit 函数来触发父组件的事件, ``this.$emit('delete',this.index);``, 其中this.index作为参数传递给父组件的回调函数, 这之前已经在组件的props中加入index参数, index的作用是表示这是第几个元素,方便删除;
+ 最后只要在父组件中, 也就是vue对象的methods中加上delete的回调函数handleItemDelete来删除该条目即可.

### 六、完整代码

```javascript
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>TodoList2</title>
	<script type="text/javascript" src="./vue.js"></script>
</head>
<body>
	<div id="app">
		<input type="text" v-model='inputValue'>
		<button v-on:click='handleBtnClick'>提交</button>
		<ul>
			<!-- <li v-for="item in list">{{ item }}</li> -->
			<todo-item v-for='(item,index) in list'
						v-bind:content='item'
						v-bind:index='index'
						@delete='handleItemDelete'>
			</todo-item>
		</ul>
	</div>
	
	<script>
		// Vue.component('TodoItem',{
		// 	props: ['content'],
		// 	template: "<li>{{ content }}</li>"
		// })
		var TodoItem = {
			props: ['content','index'],
			template: "<li @click='handleItemClick'>{{ content }}</li>",
			methods: {
				handleItemClick: function() {
					this.$emit('delete',this.index);
				}
			}
		}
		var app = new Vue({
			el: '#app',
			components: {
				TodoItem: TodoItem
			},
			data: {
				list: [],
				inputValue: ''
			},
			methods: {
				handleBtnClick: function() {
					this.list.push(this.inputValue);
					this.inputValue = '';
				},
				handleItemDelete: function(index) {
					this.list.splice(index,1);
				}
			}
		})
	</script>
</body>
</html>
```

