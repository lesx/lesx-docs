# 基于React开发范式的思考：写在Lesx发布之际

例子：[lesx-example](https://github.com/lesx/lesx-example)  
webpack loader: [lesx-loader](https://github.com/lesx/lesx-loader)

## 一些背景

现在前端框架已经呈现出React、Angular、Vue三足鼎立的局势，对于三者的对比以及技术选型的思考与争论也被讨论了非常多，比如知乎上的这个问题：[react.js,angular.js,vue.js学习哪个好？](https://www.zhihu.com/question/39943474)，对于这个问题我们不再做过多赘述。但不管怎么样，现在github上star数最多、npm上安装量最大的还是`React`，阿里巴巴很多团队的技术栈也是基于React的。此篇文章也是基于`React`的开发范式来进行讨论的。  

JSX的模板范式没有选择HTML模板，而是完全基于JS的，同时提供了一种JSX的语法糖，方便用户的开发。这样做是有几种考虑的，首先React是跨平台跨终端的，不仅可以在Web browser中运行，还可以基于RN在移动端APP、服务端基于SSR来运行，基于虚拟DOM的实现让他可以轻松地做到以上几点，另外，完全基于JS的开发可以不用掌握类似Vue/angular的指令式的语法，而是更多的偏向于使用纯js的语法开发范式，一次学习终身受益，而不用每次在开发的过程中还要去查看API文档。  

但是，React的这种开发模式也带来了一个额外的问题，就是jQuery时代尊崇的`UI与逻辑分离`的最佳实践在JSX时代又有了极大的`后退`。于是我们一直在思考，能不能有一种模式，既能享受像Vue那样`UI、展示(样式)与逻辑`分离，方便维护与可扩展，又能享受React JSX的语法带来的便利呢？


## Lesx的诞生

基于上面的思考，于是有了Lesx这个`构建式的框架`。  

`构建式的框架`并不是我们的首创，但是这个概念不知道是不是我们第一次正式提出来。业界已经有的AOT(Ahead Of Time)、非侵入式的框架比较知名的是[svelte](https://www.npmjs.com/package/svelte)，他的开发范式跟Lesx比较相似，但是他并不是基于React或者哪个框架的，而是自己研发了一套底层组件机制，对于模板代码的解析也是基于自己实现的一套AST解析实现，语法类似于Handlebars。基于React的开发范式跟Lesx比较相似的还是[react-templates](https://www.npmjs.com/package/react-templates)，他称自己是：`Lightweight templates for React`。他只是把React Class的render部分抽了出来，DSL会被编译成`React.createElement`，然后生成一个函数作为React Class的render方法。同时，react-templates里还增添了很多类似vue的指令的功能，比如：`rt-if`，`rt-repeat`等，这样的框架的问题就是问题解决的并不是很彻底，抽出render部分的同时，我们还是需要对React创建部分需要大量的代码书写；同时，对于JSX语法扩展指令的模式增添了开发者的学习成本，后面开发中也需要不断地去查看文档如何使用这些指令，这是我们极不推崇的。  

在这样的背景下，我花了两天时间，早起晚睡、憋屎憋尿的完成了基于React做到`UI、展示(样式)与逻辑`分离的构建式开发框架：`Lesx`的初版。


## 基于Lesx的开发模式

Lesx作为webpack的loader存在，使用类似Vue的单文件的开发范式，方便开发者的代码组织与开发：  

index.lesx:

```html
<style>
	a {
		color: red;
	}
</style>

<template>
	<div>
		<a onClick={this.func}>点我</a>

		{console.log(this.props)}

		<If condition={ this.props.valid }>
			<div>{this.state.name}</div>
		</If>

		<Button type="primary" onClick={() => {
			alert('I am an antd button!');
			$setState({
                name: 'new name'
            });
		}}>antd button</Button>

		<My />
	</div>
</template>

<script>
	module.exports = {
		props: {
			valid: true
		},

		state: {
			name: 'xiangzhong.wxz'
		},

		func({
			setState,
		}) {
			alert('I am a function!');

			setState({
				name: 'new name'
			});
		}
	};
</script>
```

很明显的，他有几个特点：  

### UI、样式与逻辑分离

lesx文件有style/template/script三个标签，内部分别存放他们对应的内容代码。  

`style`部分我们默认使用跟css完全兼容同时有更多便利性语法的`Sass`语言，后面马上也会支持`Less`语法。

`tenplate`部分则完全是React的jsx语法，同时由以下几个扩展：

- 我们基于babel插件`jsx-control-statements`提供了便利性的控制流标签，比如：`If`，`For`等等，语法非常简单，一次学习终生高效！当然，有的同学可能会不认可这种标签扩展控制流的模式，此时你也可以继续使用你熟悉的三元运算符、数组map等方式来实现逻辑与展示控制，但是我们相信，标签控制符是更清晰、更容易维护的开发模式；

- 你可以在DSL里面使用一些辅助性全局变量：
 - `$setState`: `this.setState`的简便写法，通过改变state值来触发UI渲染；
 - `$getRef`: React通过组件ref属性获取组件的简便写法；
 - `$getProps`: 获取React属性的简便性方法，相当于：this.xxx；

后面我们还会做一些其他的更高级的便利性扩展，比如：接入[axios](https://www.npmjs.com/package/axios)的异步操作，React的forceUpdate便利性机制等等。

`script`部分是用于书写前端逻辑处理的地方，你可以使用ES6的语法，做各种的数据处理，只需要最后把一个对象交给`module.exports`变量即可，这个对象可以包含如下内容：

- `state`: React Component的state初始值，可以是对象也可以是函数；
- `props`: React props初始值，可以是对象也可以是函数；
- `React组件的生命周期钩子函数`: 比如：`componentDidMount`等，会被自动挂在到最终生成的React Component Class里面去；
- `其他任意的属性或方法`: 均会被挂在到React Component实例(this)上去，而且，对于方法部分会被自动绑定到this作用域(this.xxx.bind(this)) 。

对于异步处理部分，默认可以直接调用`this.axios.xxx`的方法来实现，并支持ES7：`async/await`语法：

```javascript
module.exports = {
	async getData(reqArg = {}) {
		const res = await this.axios.post('url/post', reqArg);

		return res;
	}
};
```

同时，支持异步请求库可配置，可以在loader的配置里配置自己的异步请求库，此时会替换掉默认的axios。但这一块功能暂时还没有加入，承诺在接下来的一周之内会加上去。目前可以通过组件props传递的方式来使用异步，比如：

```javascript
import App from './index.lesx';
import axios from 'axios';


console.log('App:', App);

render(<App
	axios={axios}
	components={{
		My,
	}}
/>, document.querySelector('#root'));
```  

然后在lesx文件的script里面就可以这样用：

```javascript
module.exports = {
	props: {
		valid: true
	},

	state: {
		id: 1001,
	},

	async getData(reqArg = {}) {
		const res = await this.props.axios.post('url/post', reqArg);

		return res;
	},

	clickHandler({
		setState,
	}) {
		const {
			id,
		} = this.state;

		const userData = this.getData({
			id,
		});

		setState({
			name: userData.name,
		});
	}
};
```

### 开发的极大便利: 

UI库是我们在开发中重度依赖的部分，特别是对于像React这种完全组件化的开发框架来说，有个好用的UI框架简直是如虎添翼，会让我们的开发效率得到极大地提升！所以，我们的开发框架默认集成了国内最优秀的React UI库：[antd](https://ant.design/index-cn)，当然了，你也可以通过loader的配置来更改UI库，比如可以使用[material-ui](http://www.material-ui.com/)等。  

在配置了UI库之后，无需做任何工作就可以直接在`template`标签里面使用该UI库的任意组件了，比如使用Button组件：

```html
<script>
	<Button type="primary" onClick={() => {
		alert('I am an antd button!');
		$setState({
            name: 'new name'
        });
	}}>antd button</Button>
</script>
```

Lesx不仅会自动帮你打包你使用到的组件，同时，还会自动帮你把组件的样式引入；另外，基于babel的插件：`babel-plugin-import`，我们做到了按需打包，只会把你用到的组件给打包进来，保证打包后的文件的最小体积。  

### 开发者不需要书写React的组件生成代码

因为我们把React Component生成的过程全部放在了AOT里实现，所以开发者无需写`React组件生成、UI库组件引入`的操作，其实，开发者甚至不需要知道React的存在，也甚至更不需要学习React，唯一需要做的就是在渲染js文件中做一些组件引入以及渲染执行的操作，但是就这一块的成本其实是极低的。  

目前前端的资源是极度缺乏的，`整个互联网都缺前端`，所以，我们在考虑如何释放前端人力这个方案的时候，我们是否可以考虑如何降低前端的上手成本，让后端同学可以上手前端开发，做到网后端开发赋能呢？其实Lesx的开发范式一开始就是为这个方向考虑的，在满足降低前端开发成本、降低前端开发复杂度、提高代码可维护性的同时，也可以很方便的提供给后端，让后端同学可以轻松上手前端开发，从而达到合作共赢的状态。对于前端人手紧缺的公司可以考虑这个方案的落地，也许会起到意想不到的效果。  

同时，为了可扩展性，我们做了一些额外的处理。除了可以给Lesx DSL转成的Component传递属性然后可以在Lesx文件使用之外，当我们确实需要第三方或者自己之前基于React原生模式开发的组件需要拿过来直接使用的时候，我们提供了`components`属性，将任意的第三方组件放在conponents属性对象中，既可以直接在DSL中使用，如下：

```jsvascript
import React, { Component } from 'react';
import { render } from 'react-dom';
import My from './My';
import App from './index.lesx';


console.log('App:', App);

render(<App
	components={{
		My,
	}}
/>, document.querySelector('#root'));
```

在上面我们引入了自己开发的`My`组件，并放在了Lesx DSL转成的App组件的components属性里，于是可以在lesx文件中像下面这样使用：

```html
<style>
    { /** style代码 */ }
</style>

<template>
	<div>
		<a onClick={this.func}>点我</a>
		<My />
	</div>
</template>

<script>
    { /** 逻辑代码 */ }
</script>
```

怎么样，有没有那么一点点的打动你的心呢？^_^ 如果有的话，不妨去体验下Lesx，相信会带给你不一样的开发体验。  

例子：[lesx-example](https://github.com/lesx/lesx-example)  
webpack loader: [lesx-loader](https://github.com/lesx/lesx-loader)