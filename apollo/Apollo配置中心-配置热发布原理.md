# Apollo配置中心-配置热发布原理
## 1:当配置中心发布配置时，客户端响应流程
### 1:RemoteConfigLongPollService感知配置发布
        RemoteConfigLongPollService通过长轮训(结合Spring DeferredResult)迅速感知配置发布，其通知RemoteConfigRepository到ConfigServer拉取最新配置。
            1: 感知有配置发布 
            1
            2:通知RemoteConfigRepository同步配置 
            2

 ***此处引入RemoteConfigLongPollService长轮训个人觉得是为了实现配置推送模式，apollo并不是使用消息来实现配置发布的推送***

### 2:RemoteConfigRepository同步配置信息
  	RemoteConfigRepository相当于一个namespace。他会定期主动到ConfigServer同步配置信息。也会有RemoteConfigLongPollService触发实时向ConfigServer同步配置。
            	3           
          1:向ConfigServer同步配置数据
		4
          2:通知RepositoryChangeListener，有配置修改
		5
         3:启动定时向ConfigServer同步配置
		6
        4:结合RemoteConfigLongPollService实时向ConfigServer同步配置数据,能够实时响应ConfigServer配置发布
		7

###思考： 
***当RemoteConfigLongPollService感知到由配置发布时，他会将消息ID发送给RemoteConfigRepository由其发起同步消息内容的请求。此处为何不返回配置内容而是配置ID？***
 <br/>
 在github上咨询过作者，总结的答案是
 
 ```
  	1:实现幂等 
  	2:逻辑清晰  RemoteConfigLongPollService只负责感知配置发布，RemoteConfigRepository负责同步数据
 ```
              

### 3:RepositoryChangeListener、ConfigChangeListener负责处理配置发布事件

```
 二者主要做的事情是
 	1:过滤出修改过的配置
 	2:更新修改配置的bean,
个人觉得二者可以合并，可以不用分的这么细。可能是作者为了更加灵活的配置。
```
    

4:AutoUpdateConfigChangeListener.onChange()  实现配置热发布
            1:通过SpringValueRegistry 去修改拥有配置信息的bean
			8
            2:SpringValueRegistry负责管理所有需要读取配置信息的bean
			9
            3:触发bean更新配置
			10

### 总结：
```
 	1:RemoteConfigLongPollService、RemoteConfigRepository、RepositoryChangeListener、ConfigChangeListener四者层级关系在启动时由@EnableApolloConfig触发，通过集合维护下层对象。
	2:RemoteConfigLongPollService、RemoteConfigRepository内部结合线程池和Spring DeferredResult实现了配置推拉结合的模式是亮点。并没有借助于消息队列中间件。
```


## 2:客户端启动搭建监听流程
### 1:@EnableApolloConfig，启动apollo配置中心，
        具体初始化操作有ApolloConfigRegistrar完成。
### 2:PropertySourcesProcessor Spring容器预处理器
          1:从配置中心拉去配置到项目环境
		11
          2:添加配置修改监听器，使得配置中心实现热发布
		12
    3:ApolloAnnotationProcessor Spring容器预处理器
         1:完成ApolloConfig标注属性的赋值
         2:完成ApolloConfigChangeListener标注的配置更改监听事件
		13
    4:SpringValueProcessor Spring容器预处理器
        将需要动态配置的bean由SpringValueRegistry统一管理，以便配置实现热发布时，可以动态的更改配置。
		14
    5:SpringValueDefinitionProcessor 处理有xml的配置类
    6:ApolloProcessor 处理json To Object
    总结：
        以上操作结合Spring容器的高级特性，比如预处理器BeanPostProcessor实现Client读取配置和监听配置更改。

## 3:ConfigServer响应AdminServer发布配置
## 4:ConfigServer处理Client同步配置