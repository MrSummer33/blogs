# Spring Cloud Zuul
## 1:使用
### 1.1:Zuul 配置
![转发配置](https://github.com/MrSummer33/blogs/blob/master/PICTURES/SPRING-CLOUD/ZUUL/route_config.jpeg)
### 1.2:run 
```
访问网管服务端点 user/hello
自动转发到 http://localhost:18090/hello
```
SO EASY

## 2:深入解析
### 2.1:整体架构
Zuul讲整个转发流程分为三部分

```
pre:转发前
route:转发期间
post:转发后
```
![总提架构](https://github.com/MrSummer33/blogs/blob/master/PICTURES/SPRING-CLOUD/ZUUL/总流程.png)
#### 2.1.1 PRE 阶段
* 包装请求信息，例如添加用户信息、计算路由地址等
* 权限校验
* 请求统计

#### 2.1.2 ROUTE 阶段
* 转发请求

#### 2.1.3 POST 阶段
* 包装response
* 请求统计

### 2.2 内部原理
Zuul本身是一个 ***JavaWeb*** 服务。也是通过 ***Servlet*** 处理请求,只不过 ***Zuul*** 是做转发而不是真实处理。
</br>
![ZuulServlet](https://github.com/MrSummer33/blogs/blob/master/PICTURES/SPRING-CLOUD/ZUUL/ZuulServlet.jpeg) 
</br>
Netflix Zuul原声可不介入SpringMVC，但是SpringCloud默认继承了SpringMVC
</br>
![ZuulServlet](https://github.com/MrSummer33/blogs/blob/master/PICTURES/SPRING-CLOUD/ZUUL/Dispatcher入口.jpeg) 

## 3: 过滤器 ZuulFilter
Zuul网管服务器的核心就是过滤器
### 3.1 PRE 过滤器
PreDecorationFilter:负责计算路由服务器,确定转发地址

```
该过滤器是路由转发的核心，其根据路由转发配置ZuulProperties。计算当前路径转发的目标Origin。
```
![PreDecorationFilter](https://github.com/MrSummer33/blogs/blob/master/PICTURES/SPRING-CLOUD/ZUUL/PreDecorationFilter.jpeg)

### 3.2 ROUTE 过滤器
RibbonRouteFilter:根据注册中心ServiceID转发请求,获取response
</br>
![RibbonRouteFilter](https://github.com/MrSummer33/blogs/blob/master/PICTURES/SPRING-CLOUD/ZUUL/RibbonRouteFilter.jpeg) 
</br>
SimpleHostRouteFilter:根据IP地址转发请求,获取response
</br>
![SimpleHostRouteFilter](https://github.com/MrSummer33/blogs/blob/master/PICTURES/SPRING-CLOUD/ZUUL/SimpleHostRouteFilter.jpeg) 

### 3.3 POST 过滤器
SendResponseFilter:包装response
</br>
![SendResponseFilter](https://github.com/MrSummer33/blogs/blob/master/PICTURES/SPRING-CLOUD/ZUUL/SendResponseFilter.jpeg) 

### 3.4 过滤器间通信
各个过滤器件不可直接通信,需通过改变***RequestContext***(粗略理解为当前请求)状态来通信。
</br>
整个转发过程都是对***RequestContext***对象的操作。

## 4:流程
```
	1:ZuulServlet拦截请求
	2:PRE过滤器对请求进行包装
	3:ROUTE过滤器转发请求到Origin,处理请求，得到response。
	4:POST过滤器加工RESPONSE
	5:Zuul返回Response給客户端
```

## 5:总结
网管服务器一般用作统一暴露出去端口,可避免业务服务直接暴露。充当门面的角色。
### 5.1 作用
```
1:路由:转发请求到目标Origin
2:权限校验:统一做权限校验,使得业务服务只关注与业务逻辑
3:请求统计/监控:统计请求访问情况
```
### 5.2 要求
```
1:高可用:作为唯一暴露的端口，必须承受得起高并发情况。
```
