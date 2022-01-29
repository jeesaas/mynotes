## 从零开始手把手撸码搭建前后端分离应用系列四：Axios二次封装

### 4.1节：环境隔离

在之前的章节中有提到我们在开发的过程中，通常会涉及到多环境的情况，那么，为了更好的将环境隔离开，通常，我们会定义不同的配置文件，比如：.env.dev 或者 .env.prod

- .env.dev  表示开发环境配置
- .env.prod 表示生产环境配置

我们以开发环境举例说明，先在项目根目录中创建.env.dev的文件，然后添加相关配置。

	NODE_ENV=dev
	VITE_BASE_API=http://localhost:8080
	VITE_TIME_OUT=8000

### 4.2节：Axios二次封装

在utils目录下新建request.js

	/**
	 * axios二次封装
	 */
	import axios from 'axios'
	import { ElMessage } from 'element-plus'
	import router from './../router'
	
	const TOKEN_INVALID = 'Token认证失败，请重新登录'
	const NETWORK_ERROR = '网络请求异常，请稍后重试'
	
	// 获取环境配置信息
	const env = import.meta.env
	
	const baseApi = env["VITE_BASE_API"];
	
	// 创建axios实例对象，添加全局配置
	const service = axios.create({
	    baseURL: baseApi,
	    timeout: env["VITE_TIME_OUT"]
	})
	
	// 请求拦截
	service.interceptors.request.use((req) => {
	    const headers = req.headers;
	    // 后续待完善
	    if (!headers.Authorization) headers.Authorization = 'Bearer ' + "123456";
	    return req;
	})
	
	// 响应拦截
	service.interceptors.response.use((res) => {
	    const { code, data, msg } = res.data;
	    if (code === 200) {
	        return data;
	    } else if (code === 500001) {
	        ElMessage.error(TOKEN_INVALID)
	        setTimeout(() => {
	            router.push('/login')
	        }, 1500)
	        return Promise.reject(TOKEN_INVALID)
	    } else {
	        ElMessage.error(msg || NETWORK_ERROR)
	        return Promise.reject(msg || NETWORK_ERROR)
	    }
	})
	
	/**
	 * 请求核心函数
	 * @param {*} options 请求配置
	 */
	function request(options) {
	    options.method = options.method || 'get'
	    if (options.method.toLowerCase() === 'get') {
	        options.params = options.data;
	    }
	    service.defaults.baseURL = baseApi.toString();
	    return service(options)
	}
	
	['get', 'post', 'put', 'delete', 'patch'].forEach((item) => {
	    request[item] = (url, data, options) => {
	        return request({
	            url,
	            data,
	            method: item,
	            ...options
	        })
	    }
	})
	
	export default request;

为了配合项目开发的进度，以上只是对Axios进行了一些简单的封装，后续随着项目的完善，Axios二次封装的代码也会相对应的进行调整。


### 4.3节：引入Axios封装文件

在main.js文件中加入如下两段代码：

	import request from './utils/request'
	
	app.config.globalProperties.$request = request;

完整代码如下：

	import { createApp } from 'vue'
	import App from './App.vue'
	// 引入路由文件
	import router from './router'
	import ElementPlus from 'element-plus'
	import 'element-plus/dist/index.css'
	import request from './utils/request'
	
	const app = createApp(App)
	app.config.globalProperties.$request = request;
	app.use(ElementPlus)
	// 加载路由信息
	app.use(router)
	app.mount('#app')

### 4.4节：程序调用

	this.$request({
		method:'get',
		url:'/demo',
		data:{}
	}).then((res)=>{
		console.log(res)
	})


