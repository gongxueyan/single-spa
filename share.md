# single-spa

<a name="6ljBw"></a>
## 优势
- 多框架并存
- 使用新框架，写新业务
- 懒加载

<a name="wt9wg"></a>
## 代码块

<a name="aSTcL"></a>
#### 定义一个子应用
```javascript
import {registerApplication, start} from 'single-spa';

// 始终渲染的应用，第三个参数为：true
registerApplication('navbar', 
                () => import('./navbar/navbar.app.js'), () => true);
// 声明一个根组件
registerApplication('home', () => import('./home/home.app.js'), 
                  () => location.pathname === "" || location.pathname === "/");
// 生命子应用，子应用的路径前缀
registerApplication('react', () => 
                        import('./react/react.app.js'), pathPrefix('/react'));

// 返回一个布尔值，true 或 false
function pathPrefix(prefix) {
    return function(location) {
        return location.pathname.indexOf(`${prefix}`) === 0;
    }
}

start(); // 启动应用

// react.app.js 定义
import React from 'react';
import ReactDOM from 'react-dom';
import singleSpaReact from 'single-spa-react';
import rootComponent from './root.component.js';

// 定义一个react 子应用
const reactLifecycles = singleSpaReact({
  React,
  ReactDOM,
  rootComponent,
  domElementGetter: () => document.getElementById('react-app')
});

/**
 * 类似的可以定义vue子应用
 * import singleSpaVue from 'single-spa-vue';
 * const vueLifecycles = singleSpaVue({
 * 	Vue,
 * 	appOptions: {},
 * });
 */

/**
 * 导出生命周期，以备其他方法加入钩子函数
 */
export const bootstrap = [
  reactLifecycles.bootstrap,
];

export const mount = [
  reactLifecycles.mount,
];

export const unmount = [
  reactLifecycles.unmount,
];

// root.component.js
// 根组件 和 框架通信例子
export default class Root extends React.Component {
	constructor() {
    super();
    this.state = {
      frameworkInspector: false,
    };
  }
  
  render() {
  	<div>{this.state.frameworkInspector}</div>
  }
  
  componentWillMount() {
    this.subscription = showFrameworkObservable.subscribe(newValue => this.setState({frameworkInspector: newValue}));
  }
  
  componentWillUnmount() {
    this.subscription.dispose();
  }
}
```

<a name="YUZgq"></a>
#### 模板html

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<meta name="viewport" content="width=device-width">
		<title>Single-spa examples</title>
		<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.1.1/jquery.min.js"></script>
		<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/materialize/0.97.8/css/materialize.min.css">
		<link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">
		<script src="https://cdnjs.cloudflare.com/ajax/libs/materialize/0.97.8/js/materialize.min.js"></script>
    <script src="/build/common.js"></script>
    <script src="/build/main.js"></script>
	</head>
	<body>
		<div id="navbar"></div>
		<div id="home"></div>
		<div id="react-app"></div>
		<div id="angular"></div>
		<div id="vue-app"></div>
		<div id="preact-app"></div>
	</body>
</html>
```


<a name="4zFtE"></a>
#### single-spa特点

- respond to url routing events
- 关注三个生命周期：bootstrap, mount, and unmount
- 状态
  - active：listen routing events and put content on the DOM
  - inactive：not listen routing events and totally removed from the DOM.
- single-spa-config
  - a html
  - a js register applications
    - A name
    - A function to load the application's code
    - A function that determines when the application is active/inactive （boolean）




<a name="NyBxB"></a>
### 将React项目迁移到single-spa

- set single-spa config
  - 有webpack：将single-spa config作为webpack.config.js 的 enrty
  - 劫持当前当前入口 index.js

<br />
<a name="nkW1i"></a>
#### 第一步：
从[https://github.com/alocke12992/migrating-to-single-spa-react-starter](https://github.com/alocke12992/migrating-to-single-spa-react-starter) 下载代码，将registerServiceWorker 文件，放到src目录下
<a name="GFdbf"></a>
#### 第二步
在src下新建index.js 和 root.app.js 文件

index.js 中代码如下
```javascript
import registerServiceWorker from './registerServiceWorker';
import {registerApplication, start} from 'single-spa';

registerApplication(
  'root', // Name of this single-spa application
  () => import('./root.app.js'), // async function to load application
  () => true // 匹配location.pathname 或 hash
)
start();
registerServiceWorker();
```

root.app.js 中代码如下
```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import singleSpaReact from 'single-spa-react';
import App from './containers/App.js';

const reactLifecycles = singleSpaReact({
  React,
  ReactDOM,
  rootComponent: App,
  domElementGetter,
})

export const bootstrap = [
  reactLifecycles.bootstrap,
];

export const mount = [
  reactLifecycles.mount,
];

export const unmount = [
  reactLifecycles.unmount,
];

function domElementGetter() {
  // This is where single-spa will mount our application 
  return document.getElementById("root")
}
```

