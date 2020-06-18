 

## vue中的动画特效

### 1、vue中css动画原理

　　这里先写一点keyframes的动画代码，怕忘记

```css
@keyframes bounce {
	0% {
		transform: scale(0);
	}
	50% {
		transform: scale(1.5);
	}
	100% {
		transform: scale(1);
	}
}
.fade-enter-active {
	transform-origin: left center;
	animation: bounce 1s;
}
.fade-leave-active {
	transform-origin: left center;
	animation: bounce 1s reverse;
}
```

　　vue中的css动画原理，又称是过渡原理，下面讲一下原理和使用方法。

+ 首先在要添加动画的标签外面加上<transition>标签包裹起来；
+ **添加相应的样式，其中这些样式的名字都是以及固定好了的**，这里设置的name要和后面类前面一致，若未设置，则默认为v-开头：

```html
<transition name='fade'>
	<div v-if='show'>Hello, world</div>
</transition>
```

#### 1）显示

![5-1](..\pictures\5-1.png)

+ 由上图可以清晰的看到，动画开始第一帧 自动加上fade-enter和fade-enter-active类，第二帧时将fade-enter去掉，加上fade-enter-to类，最后当动画结束时将上面两个类去掉；
+ 所以显示的时候，我们添加样式为：

```css
.fade-enter { //显示开始前一秒，还是opacity为0
	opacity: 0;
}
.fade-enter-active { //这个类全程都在，从一开始到最后监听opacity的变化，并且持续一秒
	transition: opacity 1s;
}
```

#### 2）隐藏

![5-2](..\pictures\5-2.png)

+ 和上面显示类似，就不多说

```css
.fade-leave-to { //开始的第二帧才能设为0，一开始默认为1
	opacity: 0;
}
.fade-leave-active { //全程监听opacity变化
	transition: opacity 1s;
}
```

#### 3）一点小疑惑

+ 一开始看的时候有个问题，**为什么显示和隐藏的时候都是设置opacity为0呢**？虽然原理不一样，显示是开始第一帧还要为0，隐藏是第二帧才开始为0，两个都被设置的face-leave-active监听到就加上动画；
+ 后来有点想明白了。**元素默认的opacity是为1的，所以如果要让transition感知到opacity的变化**，就得有个从1到0或从0到1的变化；
+ **显示**：在开始第一帧加上face-enter设置opacity为0，紧接着第二帧这个类就被去掉了，而默认的opacity为1，所以这个时候transition感知到了变化，之后opacity也一直为1不会发生变化，就形成了显示动画效果；
+ **隐藏**：开始第一帧默认opacity为1，从第二帧开始加上face-enter-to类，，这时候transition感知到了变化，而这个类是从那时开始一直都存在的类，所以过程中opacity也不会发生变化了直到最后动画完成都被移除，就形成了隐藏动画效果；

### 2、vue中使用animate.css库

　　自己写动画固然好，但是有时需要一些复杂动画效果的时候可能还是要借助库来实现比较好，要什么样的动画去查就好了。

[animate.css官网链接]: https://animate.style/

　　下面说一下使用中要注意的几点：

+ 使用animate.css同样也还是要遵循上面两张图的顺序，且这里**要自定义隐藏和出现的类名**，且类名中一开始要加上animated这个类，具体看下面：

```html
<transition name='fade'
				enter-active-class='animated shake'
				leave-active-class='animated bounce'
>
```

+ 如果要**第一次页面刷新完毕也显示出场动画**，就要下面这样写，添加一个appear属性，以及给appear-active-class添加和出场一样的类：

```html
<transition name='fade'
					appear
					enter-active-class='animated shake'
					leave-active-class='animated bounce'
					appear-active-class='animated shake'
					>
```

### 3、同时使用过渡和动画（前面1 2的结合）

+ 其实说同时使用，也就是在2的基础上把1也加进去即可，也就是在enter-active-class和leave的那个里面加上fade-enter-active和fade-leave-active，再在相应的类里写过渡动画；

```javascript
//过渡动画
.fade-enter,
.fade-leave-to {
	opacity: 0;
}
.fade-enter-active,
.fade-leave-active {
	transition: opacity 3s;
}
<transition name='fade'
					appear
					enter-active-class='animated shake fade-enter-active'
					leave-active-class='animated bounce fade-enter-active'
					appear-active-class='animated shake'
					>
```

+ 那同时使用的话，若动画时长不一样那到底由谁来决定呢？这里可以自己设置一下按照谁来决定，在transition标签里加上`type=transition`，这样就是按照过渡动画来的；
+ 时长也可以直接通过绑定自定义属性`:duration`来决定，这里可以直接传数字，也可以传一个对象，如`:duration='{enter: 5000, leave: 100000}'`，单位是毫秒。

### 4、vue中的js动画和velocity.js的结合

+ vue中还存在几个于动画相关的钩子，也就是运行到一定时候会自动触发的事件，我们可以给这些事件绑定处理函数来实现动画效果；
+ enter的话，before-enter、enter、after-enter，这三个是顺序被触发的，其中enter有个回调函数，调用它才会触发after-enter，下看代码：

```javascript
//绑定事件的标签
<transition 
		name='fade'
		@before-enter='handleBeforeEnter'
		@enter='handleEnter'
		@after-enter='handleAfterEnter'
>
//事件处理函数
//动画运行前一帧
//接收的el参数为transition包裹的元素，也就是要进行动画的元素
handleBeforeEnter: function(el) {
     el.style.color = 'red';
},
//动画运行中，done是回调函数
handleEnter: function(el,done) {
     setTimeout(()=>{
           el.style.color = 'pink';
           el.color='pink';
     },2000);
     setTimeout(()=>{
           done(); //这里手动调用done是为了说明动画结束了，下面才会由afterenter
     },4000);
                },
//动画运行结束
handleAfterEnter: function(el) {
     el.style.color='skyblue';
}
```

+ 隐藏动画的话和入场一样，不再赘述；

+ velocity.js的使用的话其实也差不多，只是也还是不用自己写了，先引入，再修改上面事件处理函数里的内容，见代码：

```javascript
//绑定事件的标签
<transition 
		name='fade'
		@before-enter='handleBeforeEnter'
		@enter='handleEnter'
		@after-enter='handleAfterEnter'
>
//事件处理函数
//动画运行前一帧
//接收的el参数为transition包裹的元素，也就是要进行动画的元素
handleBeforeEnter: function(el) {
     el.style.opacity = 0;
},
//动画运行中，done是回调函数
handleEnter: function(el,done) {
     Velocity(el, {
                opacity: 1
			 },{
				 duration: 1000,
				 complete: done //这是结束之后自动调用的函数
	})
},
//动画运行结束
handleAfterEnter: function(el) {
     console.log('动画结束');
}
```

### 5、vue中多个元素或组件的过渡

#### 多个元素的过渡

+ 就直接用transition标签将两个要过渡的元素包裹起来，用v-if和v-else来实现发现元素显示依然没有动画：

```javascript
.v-enter,
.v-leave-to {
	opacity: 0;
}
.v-enter-active,
.v-leave-active {
	transition: opacity 1s;
}
<transition>
	<div v-if='show'>Hello, world</div>
	<div v-else>Bye, world</div>
</transition>
```

+ 当有**相同标签名**的元素切换时，需要通过 `key` attribute 设置唯一的值来标记以让 Vue 区分它们，否则 Vue 为了效率只会替换相同标签内部的内容。即使在技术上没有必要，**给在 `<transition>` 组件中的多个元素设置 key 是一个更好的实践。`<div v-if='show' key='hello'>Hello, world</div> <div v-else key='bye'>Bye, world</div>`

+ 现在两个元素的隐藏和显示是同时进行的，可以在transition标签里添加mode属性来控制谁先谁后进行，值可以为 out-in 或是 in-out，从语义就能看出不一样；

#### 多个组件的过渡

+ 其实和上面没有什么区别，只是将transition里面两个div换成组件即可，这里用不用动态组件都可以实现（组件的知识复习上一章）。
+ 不同于 `<transition>`，它会以一个真实元素呈现：默认为一个 `<span>`。你也可以通过 `tag` attribute 更换为其他元素。
+ **过渡模式不可用**，因为我们不再相互切换特有的元素。
+ 内部元素**总是需要**提供唯一的 `key` attribute 值。
+ CSS 过渡的类将会应用在内部的元素中，而不是这个组/容器本身。

```javascript
<transition mode='out-in'>
	<component :is='type'></component>
</transition>
		Vue.component('child', {
			template: '<div>child</div>'
		})
		Vue.component('child-one', {
			template: '<div>child-one</div>'
		})
		var vm = new Vue({
			el: '#root',
			data: {
				type: 'child'
			},
			methods: {
				handleBtnClick: function() {
					this.type= (this.type==='child'?'child-one':'child');
				}
			}
		})
```

### 6、vue中的列表过渡

+ 其实列表过渡也和上面一样，只是把上面的transition标签改为transition-group即可，看代码：

```javascript
	.v-enter,
    .v-leave-to {
        opacity: 0;
    }
.v-enter-active,
    .v-leave-active {
        transition: opacity 1s;
    }
<transition-group>
	<div v-for='item of list' :key='item.id'>
		{{item.title}}
	</div>
</transition-group>
<script>
	var counter = 0;
	var vm = new Vue({
		el: '#root',
		data: {
			list: []
		},
		methods: {
			handleBtnClick: function() {
			this.list.push({
					title: 'Hello num'+counter,
					id: counter++
				})
			}
		}
	})
</script>
```

### 7、vue中的动画封装(封装在一个子组件里)

+ 写一个动画模板，封装在一个子组件里，而模板的实例内容通过插槽(slot)的方式体现，哎看代码吧解释好费劲：

```javascript
<fade :show='show'>
	<div>Hello world</div>
</fade>
//当然上面的css还是不变
Vue.component('fade',{
			props: ['show'],
			template: `
					<transition>
						<slot v-if='show'></slot>
					</transition>
					`,
			methods: {
				handleBtnClick: function() {
					this.show = !this.show;
				}
			}
		})
```

+ 若是要将动画直接封装在子组件里，就用js的动画：

```javascript
Vue.component('fade',{
			props: ['show'],
    		//记得这里的事件触发是绑定到transition上！不是插槽上！
			template: `
					<transition
							@before-enter='handleBeforeEnter'
							@enter='handleEnter'
							@after-enter='handleAfterEnter'
					>
						<slot v-if='show'></slot>
					</transition>
					`,
			methods: {
				handleBeforeEnter: function(el) {
					     el.style.color = 'red';
					},
				handleEnter: function(el,done) {
				     setTimeout(()=>{
				           el.style.color = 'pink';
				     },2000);
				     setTimeout(()=>{
				           done(); 
				     },4000);
				                },
				handleAfterEnter: function(el) {
				     el.style.color='skyblue';
				}
			}
		})
```

