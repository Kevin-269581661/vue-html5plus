# vue-html5plus

原本这只是一个极简的移动端现代浏览器dom操作js工具库，但是几经思考多次更名，最终定位为html5plus App的骨架。现在数据双向绑定已经是众多框架标配了，大势所趋，所以本骨架也支持数据双向绑定。本着站在巨人肩膀上的原则，框架整体上基于Vue 2.0改造，Vue.js推荐的开发方式是组件化构建SPA应用，这对于熟悉5+ App开发的朋友可能不是很习惯，其实基于webview的MPA应用也是一种不错的选择。本框架基于这个原则，封装了5+ App开发中一些常用操作处理方法，另外也完全可以用于组件化开发模式。本框架不会限制开发者使用何种UI框架，但是结合mui可以更快捷地进行开发，推荐大家使用配置式的写法，用这个骨架去统一代码风格，更加方便的维护项目代码。

好的框架一定是五分钟可以上手的,下面我们以这个例子说明：

```
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width,initial-scale=1,minimum-scale=1,maximum-scale=1,user-scalable=no" />
    <title>hello vue-html5plus！</title>
    <link rel="stylesheet" type="text/css" href="css/mui.css"/>
    <link rel="stylesheet" type="text/css" href="css/app.css"/>
</head>
<body>

	<div id="app">
		<header class="mui-bar mui-bar-nav">
			<button class="mui-btn mui-btn-link">
				<span class="name">{{city}}</span>
				<span class="mui-icon mui-icon-arrowdown"></span>
			</button>
		    <h1 class="mui-title">{{title}}</h1>
		</header>
		<div class="mui-content mui-content-padded">
			<h3 class="mui-subtitle">定位：</h3>
		    {{position}}
		    <h3 class="mui-subtitle">网络：</h3>
		    {{networktype}}
		    <h3 class="mui-subtitle">ajax：</h3>
		    {{getData}}
		    <h3 class="mui-subtitle">爬虫：</h3>
		    <button type="button" class="mui-btn mui-btn-blue mui-btn-block" v-on:tap="parser">爬虫实例</button>
		</div>
	</div>

	<script src="html5plus://ready"></script>
	<script src="js/mui.min.js"></script>
	<script src="js/vue.js"></script>
	<script src="js/vue-html5plus.js"></script>
	<script type="text/javascript">
		VuePlus({
			el: '#app',
			data: {
				title: 'hello vue-html5plus!',
				city: '',
				position: '',
				networktype: '',
				getData: ''
			},
			methods: {
				parser: function () {
					mui.openWindow('example/parser.html');
				}
			},
			mounted: function () {
				console.log('mounted:'+JSON.stringify(this.$data))
			},
			domReady: function () {
				console.log('domReady:'+JSON.stringify(this.$data))
				console.log(vp('.mui-title').html());
			},
			plusReady: {
				init: function () {
					console.log('plusReady');

					var self = this;
					vp.get('http://httpbin.org/get').then(function(response){
						self.getData = response.body;
					}).catch(function(error){
						console.log(JSON.stringify(error));
					})
				},
				getLocation: function (response) {
					this.position = response.message;
					this.city = response.message.address.city;
				},
				getNetworkType: function (response) {
					this.networktype = response;
				}
			}
		})
	</script>
</body>
</html>
```

### VuePlus(config)初始化

config 可以是一个json对象，也可以是一个字符串。当config是对象时，是作为页面初始化时使用，当config是字符串时，用作选择器，用法同jQuery一致，可以链式调用进行必要的DOM操作，虽然这些方法完全可以实现原生实现，但是为了方便新手从jQuery向Vue过渡进行的封装，只封装了几个最常用的方法。

由于本骨架是基于Vue.js,如果你是新手，对vue.js一脸懵逼，推荐先看看[Vue.js指南](http://cn.vuejs.org/v2/guide/index.html)。骨架未对vue.js进行任何改造，只是将vuejs封装进VuePlus骨架中，将Vue实例对象挂载在VuePlus对象上，可以通过VuePlus.vm获取Vue实例对象，从而和vue.js中进行一样的操作，如进行数据绑定，可使用VuePlus.vm.* 或 VuePlus.vm.$data.* 或更简单方便的this.* ，这里完全是为了绑定数据的时候更加方便，将this指向为Vue实例对象，所以我们可以在domReady和plusReady下使用this来调用data下的数据，详细可以参考：[Vue 实例](http://cn.vuejs.org/v2/guide/instance.html)。另外需要特别强调的是如果没有引用Vue.js或config中没有el属性，默认没有初始化Vue,此时domReady和plusReady下this指向VuePlus对象。

### 生命周期事件

Vue中不同的钩子在实例生命周期的不同阶段调用，如created、 mounted、 updated 、destroyed，Vue2.0中去掉了ready事件，而是使用mounted代替。mounted会比domReady和plusReady先执行，所以如果需要尽快对页面DOM进行操作，可以在mounted中进行；在5+ App中，我们最关心的是domReady和plusReady，如果在页面添加了`<script src="html5plus://ready"></script>`，将5+ API提前注入，则plusReady会先于domReady执行，否则相反domReady先执行，为了确保页面完全加载才能执行的方法我们可以放在domReady钩子中，当然直接放在plusReady中也是可以的，只是如果我们希望代码更加清晰整洁，最好是将plus相关的操作单独出来，这样也方便维护多个版本。推荐在首页和浏览器模板页使用5+ API提前注入，其他页面保持默认的顺序最合适。

### 选项卡实例

```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title></title>
		<meta name="viewport" content="width=device-width, initial-scale=1,maximum-scale=1,user-scalable=no">
		<meta name="apple-mobile-web-app-capable" content="yes">
		<meta name="apple-mobile-web-app-status-bar-style" content="black">
		<link rel="stylesheet" href="css/mui.min.css">
		<link rel="stylesheet" type="text/css" href="css/app.css"/>
		<style>
			.mui-tab-item .mui-tab-label{
				font-family: "PingFangSC-Regular","microsoft yahei";
				color: #666666;
			}
			.mui-tab-item.mui-active .mui-tab-label{
				color: #FF4900;
			}
			.mui-bar-tab .mui-tab-item.mui-active{
				color: #ff4900;
			}
		</style>
	</head>

	<body>
		<div id="app">
			<header class="mui-bar mui-bar-nav">
				<button class="mui-btn mui-btn-link" v-if="activeIndex==0">
					<span class="name">{{city}}</span>
					<span class="mui-icon mui-icon-arrowdown"></span>
				</button>
				<h1 class="mui-title">{{title[activeIndex]}}</h1>
			</header>

			<nav class="mui-bar mui-bar-tab">
				<a class="mui-tab-item" v-bind:class="{'mui-active': (index === 0)}" v-for='(item, index) in tabbar' v-on:tap='changeView(index)'>
					<span class="mui-icon" v-bind:class="item.icon"></span>
	        <span class="mui-tab-label">{{item.name}}</span>
				</a>
			</nav>
			<div class="mui-content"></div>
		</div>

		<script src="html5plus://ready"></script>
		<script src="js/vue.js"></script>
		<script src="js/vue-html5plus.js"></script>
		<script src="js/mui.min.js"></script>
		<script src="js/tabbar.js"></script>
		<script type="text/javascript">
			tabbar.init({
				activeIndex: 0,
				style: {
					top: '44px',
					bottom: '51px'
				},
				subpages: ['example/main-home.html', 'example/main-parser.html', 'example/main-test.html']
			})

			VuePlus({
				el: '#app',
				data: {
					city: '',
					title: ['hello Vue-html5plus！', '爬虫实例', '测试'],
					activeIndex: 0,
					tabbar: [
						{name: '首页', icon: 'mui-icon-home'},
						{name: '爬虫', icon: 'mui-icon-navigate'},
						{name: '测试', icon: 'mui-icon-gear'}
					]
				},
				methods: {
					changeView: function (index) {
						tabbar.switchView(index, function(){
							console.log(index);
						});
					}
				},
				plusReady: {
					getLocation: function (response) {
	            this.city = response.message.address.city.replace('市', '');
	        }
				}
			});
		</script>
	</body>
</html>
```

### 浏览器模板

很多应用场景是我们需要加载网络地址，实现一个简易的内置浏览器，下面的例子是一个简单的例子：

```
<html>
	<head>
		<title></title>
		<meta charset="UTF-8"/>
		<meta name="viewport" content="width=device-width,initial-scale=1,minimum-scale=1,maximum-scale=1,user-scalable=no" />
		<link rel="stylesheet" type="text/css" href="../css/mui.min.css"/>
		<link rel="stylesheet" type="text/css" href="../css/app.css"/>
		<link rel="stylesheet" type="text/css" href="../css/progressbar.css"/>
	</head>
	<body>
		<header class="mui-bar mui-bar-nav">
		    <a class="mui-action-back mui-icon mui-icon-left-nav mui-pull-left"></a>
		    <h1 class="mui-title"></h1>
		</header>
		<div class="progressbar"><span></span></div>

		<script src="../js/vue-html5plus.js"></script>
		<script src="../js/progressbar.js"></script>
		<script type="text/javascript">
			VuePlus({
				browser: {
					url: function() {
						return 'https://www.baidu.com/';
					},
					bounce: false,
					style: {
						top: '44px',
		        bottom: '0px'
					},
					title: function (title) {
						var title = title.split(' - ')[0];
						vp('.mui-title').html(title);
					},
					loading: function () {
						console.log('loading');
						progressbar.loading();
					},
					loaded: function () {
						console.log('loaded');
						progressbar.loaded();
					},
					back: function () {
						console.log('back');
					},
					close: function () {
						console.log('browser close');
					}
				}
			})
		</script>
	</body>
</html>
```

- url 为必须配置字段，支持字符串和函数，即初始化入口地址；
- bounce 设置是否启用回弹，默认是开启
- style 设置浏览器窗口样式
- title 默认需要用户设置，可以为需要设置title的选择器或者回调函数
- loading，loaded，back，close为回调函数
