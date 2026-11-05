router
安装：npm install vue-router
使用：createRouter 创建一个router对象，通过`createApp(App).use(router)`将其注册为插件
需要给router对象增加history属性，值通过使用createWebHistory()`获得
相关：userRouter()获取router的相关操作；userRoute()获取router的各种信息（例如路径，参数）
vuex
安装：npm install vuex
使用：createStore 创建对象，其他跟router相似
属性：
	1. state 
		说明：	状态存储
		使用：通过computed计算属性返回某个状态
		
	2. mutations 
		  说明：改变store中的状态的唯一方式，提交mutation
		  使用：store.commit(mutationName, payload)
	
	3. getters 
		说明：对状态进行一些操作，只需要获取到派生出的一些状态
		使用：store.getters[params]
		传参：通过返回一个函数来传参
			`(state) => (id) => {return }` 			
	4. action
		说明：- Action 提交的是 mutation，而不是直接变更状态。
			- Action 可以包含任意异步操作，mutation只能是同步的操作
		使用：store.dispatch(actionName)	