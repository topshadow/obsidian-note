* 应用的加载
	* 预渲染
	* 预加载
	* 惰性加载
	* 延迟单例
* 应用的保活
	* 单例
	* 多例
应用的声明周期
* 微服务的加载 预渲染,预加载,惰性加载,延迟单例


>有些app最好不改代码也能有插件加载



* 与主应用中组件的交互方式有以下,呼出组件(模态框),内嵌组件(div数据),
 * 与主应用可以有几大命名空间
 - [ ] permission
 - [ ] bussness(calender)
- [ ] native插件接口原生功能

应用管理使用拖拽,消息中心使用promise

应用堆栈,例如，当用户在a应用中申请打开某人的电子病历,并获得权限后查阅电子病历，此时电子病历覆盖当前应用，当退出查阅电子病历后,应用激活当前电子病历