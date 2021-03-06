# auth-system
## 简介
这是使用了springboot,shiro,mybatis-plus,redis等开发的一个简单的权限项目(目前完成了一些通用的封装 尚未全部完成)

## 项目结构介绍
```
.
├── common (一些通用的内容 不会涉及实现)
│   ├── anno 所有的自定义注解类
│   ├── buider 全局对象的构造器类
│   ├── constant 全局的常量类
│   ├── context 一些上下文对象 
│   ├── entity 全局的实体类
│   ├── enums 全局的基础枚举
│   └── exeception 和异常相关的一些基础类
├── controller(控制器)
├── dao(数据访问层)
├── dto(数据对象)
├── function(包含一些系统职能相关的内容 包括定时任务,策略服务,过滤器,拦截器等)
│   ├── aspect 放置所有的aop切面
│   ├── basic 放置一些提供实现的基本类
│   ├── config 包含spring的一些bean配置
│   ├── filter(过滤器)
│   ├── handler(处理器)
│   ├── interceptor(拦截器)
│   ├── job(定时任务)
│   └── strategy(策略相关)
│       ├── enums 策略枚举
│       ├── factory 策略工厂
│       └── service 策略服务
├── properties(系统配置类)
├── service(服务层)
├── tool(业务相关的工具类)
└── util(业务无关的工具类)
```

## 数据库介绍
初始化sql详见init.sql项目引入了liquibase配置数据库启动项目将会自动执行
如果需要关闭自动执行请修改配置spring.liquibase.enabled属性为false

## 系统工具介绍
### 后台工具
#### 缓存工具
```
// 配置项目prefix 在application.yml/application.properties 工具类和注解中的实际key将加上这个值
cache.prefix=...
// 普通工具类(注入使用)
@Autowired
RedisUtils redisUtils;
//设置缓存且不超时
redisUtils.set(key,object);
//设置缓存且自定义超时时间(/分钟)
redisUtils.set(key,object,ttlMinutes);
//设置缓存且自定义超时时间和时间单位
redisUtils.set(key,object,ttl,timeUnit);
//获取对象 什么类型存入就以什么类型取出来
redisUtils.get(key);
 
// 缓存切面(任意方法上使用,key支持SpEL表达式) 
// 此例将为该方法返回值设置10分钟的key为 fileCache-[key]的缓存,即接下来10分钟从缓存获取方法返回值。
// 注: 当切面方法返回值为 Optional,Map,List类型时 内容不存在不会存储缓存
@Cache(key = "'fileCache-' + #key",ttlMinutes = 10)
public List<UserDTO> queryList(String key)
```
#### 用户上下文注入
```
//方法拦截器将会为任意带CurrentUser注解的UserDTO对象注入获取到的用户上下文
public UserDTO getUser(@CurrentUser UserDTO userDTO)
```
#### 策略模式
提供的策略实现包含三部分内容:策略工厂，策略枚举和策略服务 工厂和服务需要使用同样的枚举去标记以便将相同类型的服务聚合到同个工厂下
工厂声明 策略使用时主要是注入工厂根据枚举值获取服务:
```java
@Component
public class TestStrategyFactory extends BaseStrategyFactory<TestStrategyEnum> {}
```
枚举声明 枚举的码值需要和spring的bean名称相同:
```java
public enum TestStrategyEnum implements BaseStrategyEnum {
    /**
     * 枚举信息
     */
    SERVICE1(TestService1Impl.class, "测试策略1"), SERVICE2(TestService2Impl.class, "测试策略2");

    TestStrategyEnum(Class<? extends BaseStrategyService> clazz, String strategyDesc) {
        this.value = SpringUtils.getBeanNameByType(clazz);
        this.strategyDesc = strategyDesc;
    }

    /**
     * 策略值
     */
    private String value;
    /**
     * 策略描述
     */
    private String strategyDesc;

    @Override
    public String getValue() {
        return value;
    }

    @Override
    public String getStrategyDesc() {
        return strategyDesc;
    }
}
```
服务声明 服务提供了同一个接口,如果需要返回值则实现BaseStrategyAndReturnService增加返回值的泛型参数 BaseStrategyService不需要返回值(但由于java的泛型约束 还是需要返回null):
```java
@Service
@Slf4j
public class TestService1Impl implements BaseStrategyService<TestStrategyEnum> {
    @Override
    public Void exec() {
        log.info("service1");
        return null;
    }
}
```
使用:
```java
public class StrategyFactoryTest extends BaseTest {
    @Autowired
    private TestStrategyFactory testStrategyFactory;

    @Test
    public void testService() {
        testStrategyFactory.getByType(TestStrategyEnum.SERVICE1).exec();
    }
}
```

### 前端工具

#### tableServer.js
基于amaze-ui data-tables的一层封装详见文档:
[tableServer](http://note.youdao.com/noteshare?id=d1831a1a33f68265315104999e5d8085&sub=9EF2DC58A15C42E78290038D73133345)

#### operateServer.js

#### strTool.js

#### optional.js