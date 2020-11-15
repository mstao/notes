# 功能

![1554079563518](../assets/1554079563518.png)

Session 分布式管理，

限流服务为 Redis 数据结构中的一种应用



秒杀辅助服务：

包括验证码、

动态 URL、

预减库存、

终止标记





缓存服务：

页面缓存： 缓存热点的前几个列表页面

URL 缓存： 缓存热点的多个商品，如小米手机；

对象缓存： 缓存最终生成的订单，加快秒杀接口、队列处理、结果查询的速度；

## *登录

![1554079059234](../assets/1554079059234.png)

**&& 底层数据模型**

① password, salt： 通过加随机 salt 防止脱库密码明文泄漏

```java
class SeckillUser {
	Long	id;
	String	nickname;
	String	password;
	String	salt;
	String	head;
	Date	registerDate;
	Date	lastLoginDate;
	Integer	loginCount;
}
```

**（一） JSR 303 校验**

自定义注解配合对应的支持类实现；



JSR 303

@Valid

@NotNull

@Length(min = 32)

@Pattern

**自定义校验器：**  **javax.validation.Constraint**

ConstraintValidator 中 Class<? extends ConstraintValidator<?, ?>>[] validatedBy();




**（二） 两次加密**

前台加密防止 HTTP 明文传输被捕获破解，从而获取到密码；

后台进行加密，防止数据库脱库，之后使用彩虹表破解；



实现：

第一次，通过对 SALT 常量进行 char 拆分，前台使用的是固定的 SALT，之后进行前后的组装；

第二次，通过为每个用户生成随机的 SALT 并入库，即使知道了 SALT 也不知道对应的截取规则从而无法进行彩虹表比对获得；



@Q: 为何要进行两次 MD5 加密？两次的理由是什么？如何实现？
HTTP 在网络中已明文传输，使用彩虹表轻易破解。
第一次防止网络传输中被截取破解，第二次防止数据库被盗取使用彩虹表破解。

第一次： 使用固定的 salt 按照字符拆分进行前后组装。  ==>> 防止 http 明文传输被截获破解
第二次： 创建随机 SALT，并按照一定规则填充并入库。      ==>> 防止 db 被破解



**（三） 分布式 Session**

根据传递过来的 token 唯一定位到该用户

（1） 提供两种方案

移动端通过 URL 重写来传入 token；

客户端通过传入的 Cookie 来获取对应的 Session；

有效兼容不同的平台实现；



（2）Redis 构建分布式 Session 的原理

session 没有存放到 tomcat 容器中，而是存放到单独的缓存中，实现使用单独的 redis 来管理 session，即为分布式 session 。 这样就保证了 app server 是一个无状态的 server，从而将 session 分离出来。



**（3） 维护有效期**

方式一： 可以在 Filter 中拦截进行重置有效期

方式二： 本系统采用 SpringMVC 拦截器来进行 Cookie、Session 的重置

每次访问后进行 redis cache 以及 cookie 的失效时间重新分配为原来的初始时间间隔。


**（4） 进一步优化(参数注入)**

优化获取用户  Session，借助 SpringMVC 的参数绑定器配合 ThreadLocal 将其直接绑定到方法参数上。


分布式 session ： 从 redis 中获得带有 expires 的token
每次访问自动更新 Cookie 过期时间


**注：  问道这块的时候说 ThreadLocal，SpringMVC 的参数绑定，拦截器实现 Session 的有效期重置**

同时为了高可用性，需要构建 Redis 的集群实现；


**&& 修改密码**

在修改密码情况下必须要输入原来的密码，防止横向越权；

涉及到缓存更新测策略；


拦截参数

Filter ⇒  DispatcherServlet  ⇒ Interceptor  ⇒  HandlerMethodArgumentResolver
                                   ^ set user to threadLocal



**&& 拦截器处理逻辑**

（1） SessionInterceptor    

维护 Session 的有效期，将 Session 放到 ThreadLocal 中进行处理，同时清理用户防止 ThreadLocal 内存泄漏；


**（2） AccessInterceptor** 

访问拦截 ，基于注解  `@AccessLimit`  中的参数来进行对应的各种处理操作。


注解中的三个参数：  

- needLogin: 设置是否需要登录, 默认情况下需要登录  

- seconds: 指定时间内  

- maxCount: 最大允许的访问数量


原理:  通过 Redis 中设定指定过期键来实现  



**（3） UserArgumentResolver**

> 分布式 Session 注入到方法参数中  

用户参数拦截器   
获取参数并直接放到请求参数中



## 秒杀辅助

**&& 秒杀前的辅助**

**（1） 数学公式验证码**

语义为某个用户对于某个商品的验证码结果

```sql
SeckillKey:verifyCode:18852860007,5          7
```


**（2） 动态 URL** 
语义： 存放某个用户对于某件商品的 URL 访问资格，资格为一个字符串。

```sql
SeckillKey:seckillPath:18852860000:2			dsjfldsjklfsjdlfjs
```


**（3） 接口限流**

语义： 某个用户访问某个 URL 的次数

在进入秒杀接口真正执行逻辑时，作为该接口的第一层校验，需要制定用户与商品，精度较高，也可防止防刷；

```
SeckillKey:
```


（4） 缓存秒杀列表页

可以尝试缓存热点的前几页放入，当前采用定期删除策略处理

```
GoodsKey:goodslist					HTML
```


（5） 页面静态化

防止秒杀中大量的流量，从而让页面只需要局部刷新部分数据即可，主体的 HTML 缓存在用户端；



**&& 秒杀中和秒杀后的辅助**

**（1） 预减库存**
语义： 某个商品当前的库存

不需要与 DB 中完全保全一致，最后可能出现秒杀的产品没有全部卖完的情况，但不会出现超卖问题。

```
GoodsKey:goodsStock:102                     28
```


**（2） 保存商品是否有库存**

语义： 某个商品当前是否有库存


作用： 通过在 Redis 中放入对应商品是否还有库存来直接返回结果，用户无需等待即可。

可能用户请求的秒杀正在排队中，此时通过此标记查询秒杀结果就不需要等待队列执行，直接返回即可。

通过判断是否有该 KEY 来实现的；


访问的流程：

① 该字段交给 seckill 核心逻辑处理，在减库存失败后意味着无库存，设置 KEY 处理；

② 之后用户异步调用查询结果时，在无该订单的情况下，通过判断对应的商品是否还有库存直接返回，无需等待处理的结果；


```
SeckillKey:goodsOver:2				true
```


（3） 缓存秒杀成功后的订单对象

在秒杀逻辑、队列逻辑以及查询中减少访问 DB，直接从缓存读取，速度更快；




## *秒杀业务

Redis 预减库存减少数据库访问；

内存标记减少 Redis 访问；

先入队缓冲，异步下单，增强用户体验；

RabbiitMQ 安装与 Spring Boot 集群；

Nginx 水平扩展；

分库分表 (MyCat)；



底层结构： 单独从原来的 Goods 中抽离处一张表作为秒杀商品，对应的从 Order_info 中抽离出一张表作为 Seckill_order。

对于以后的 9/9 包邮，各种秒杀活动都通过此方式实现。



**&& 前端控制**

倒计时，由客户端的 JS 脚本控制；

秒杀开始的时间交给 Server 端进行控制；


安全性：
原子操作，需要事务的支持，订单的生成上采用事务控制



@Q: 如何处理超卖？
当前主要针对单个数据库的处理。
通过 DB 中设置 stock > 0 ，无法保证，解决一个用户秒杀到两个商品， 借助 DB 中的唯一索引实现，之后借助事务机制实现回滚实现。

减库存控制
创建订单控制

借助 Redis 实现的分布式锁


@Q: 如何控制一个用户只秒杀到一个商品？

利用 DB 中的唯一索引字段控制；

用户可以请求多次，但是对一个商品只能够进行一次秒杀，秒杀的数量可以随意。



**&& 底层数据模型**

（1） 秒杀商品

① seckillPrice： 打折后的超低价

② stockCount： 避免超卖，Redis 预减，内存标记控制；

③ startDate,endDate： 控制可以获取动态 URL 的时间

```java
SeckillGoods {
	Long		id;
	Long		goodsId;
	BigDecimal	seckillPrice;
	Integer		stockCount;
	Date		startDate;
	Date		endDate;
}
```


（2） 秒杀订单

① (userId, goodsId) ： 为唯一索引，用来防止超卖；

② orderId： 可能有某个用户对于多件秒杀的商品仅生成一个订单；

```java
class SeckillOrder {
    Long id;
    Long userId;
    Long orderId;
    Long goodsId;
}
```


**&& 后台逻辑**

**（1） 路径校验**

校验路径，如果正确，直接从 Redis 中删除，避免不必要的内存占用开销；

某个用户对于某件商品的 URL 权限是否与我们存放的 URL 权限相同，此处权限代表一个字符串；

路径正确之后进行对应的删除，使其不影响限流的服务(注: 原来项目中无此逻辑)；



**（2） 内存标记**

通过 HashMap 存放 KEY--商品 ID， VALUE--库存是否还有；

在预减库存后进行可能的更新；

当前作用域为访问 SeckillController 的成员变量；


算是一种在数据结构选择上的优化，HashMap 在初始化之后不存在 put 操作，从而不会有扩容问题，同时对于 HashMap 中值的变更，不依赖于之前的值，且对于每个键，只需要执行一次即可，不存在 Value 进行来回变化的情况。



**（3） 预减库存**

自带了当前商品是否秒杀结束的逻辑，每次在判断业务逻辑之前进行此次预减库存，该库存与 DB 中的库存数不一致，最终可能出现卖不出的情况，但秒杀的概率本来就小，也处于可接受的范围内。


**（4） 逻辑判断**

查询当前用户是否已经秒杀到该商品，若之前已经秒杀过了，此时即是从 Redis 缓存中取出来的，避免了一次查询 DB 的操作；



**（5） 消息入队**

将封装好的 {user, goodsId} 发送到 RabbitMQ 中，由队列进行异步处理实现最后的秒杀逻辑；


## 商品模块

（1） 普通的商品表

① goodsPrice： 商品的原价，作为与秒杀商品同时展示

② goodsStock： 可对商品库存构建库存中心，秒杀商品从中提取出要秒杀的库存数量

```java
class Goods {
	Long		id;
	String		goodsName;
	String		goodsTitle;
	String		goodsImg;
	String		goodsDetail;
	BigDecimal	goodsPrice;
	Integer		goodsStock;
}
```

（2） 秒杀商品

① seckillPrice： 秒杀价，与原来价格作为对比，一般相差较大；

② stockCount： 库存数量，秒杀结束后可能任然有库存；

③ startDate|endDate： 用于控制商品是否开始了秒杀，针对到每个商品的时间粒度；

```java
class SeckillGoods {
	Long		id;
	Long		goodsId;
	BigDecimal	seckillPrice;
	Integer		stockCount;
	Date		startDate;
	Date		endDate;
}
```


**&& 前台展示数据**

（1）  基础的展示秒杀商品的对象

```java
class SeckillGoodsVO extends Goods {
	BigDecimal	seckillPrice;   
	Integer		stockCount;
	Date		startDate;
	Date		endDate;
}
```



（2）  

```java
public class GoodsDetailVO {
	int            seckillStatus	= Const.SeckillStatusEnum.NOT_BEGIN.getCode();
	int            remainSeconds	= 0;
	SeckillUser    user;
	SeckillGoodsVO goods;
}
```



（3） 订单商品信息

所属的订单信息，order；

与该商品相关的信息， SeckillGoodsVO；

```java
class OrderDetailVO {
	SeckillGoodsVO goods;
	OrderInfo order;
}
```



**&& 接口**

（1）获取商品列表

>  缓存整个 HTML

进行数据协商，表明返回的是什么格式的数据

```java
@RequestMapping(value="to_list", produces="text/html;charset=utf-8")
```



获取引擎进行加载实现：

① 获取 ApplicationContext，ThymeleafViewResolver；

② 组装 Modle 参数；

③ 获取 response, request, servletContext, local 作为 SpringWebContext 的参数；

④ 使用视图解析器指定需要解析到的视图文件，同时传入对应的 SpringWebContext 作为参数熏染；

```java
@Autowired
ThymeleafViewResolver viewResolver;
@Autowired
ApplicationContext applicationContext;
```

```java
SpringWebContext ctx = new SpringWebContext(request,response,
				request.getServletContext(),request.getLocale(), model.asMap(), applicationContext);
html = viewResolver.getTemplateEngine().process("goods_list", ctx);
```



（2） 指定商品的详情

页面静态化，防止网络中大流量数据的传输，(注: 秒杀时像供应商租借流量的)，只进行局部刷新；



## 订单模块

（） 订单详情

组装出一个 OrderDetailVO 进行前台的展示；







# 服务

## *动态 URL | 限流

语义： 某个用户访问某个接口，在给定时间内大于一定的数量不允许其进行访问；

**（1） Redis 结构**

KEY: 某个内部 URL 下用户 A 访问该接口

VAL： 在键的过期时间内还有多少次可访问机会

```sql
AccessKey:access:/seckill/path:18852860007          1
```



**（2） 实现机制**

任意使用 @AccessLimit 修饰的方法，且填写了 maxCount , time 





**（3） 其他的拦截算法及比较**

实现上： 在 Redis 中创建一个所请求 URL 给定过期时间的 KEY，访问进入拦截器

限流的算法

计数器算法：

滑动窗口算法：

漏桶算法：

令牌桶算法： 




**&& 为什么要做隐藏秒杀接口地址？**

（1）       html是可以被右键->查看源代码，如果秒杀地址写死在源文件中，是很容易就被恶意用户拿到的，就可以被机器人利用来刷接口。

（2）       通过一个接口来返回秒杀地址的好处是，可以在活动临近开始的时候，服务端可以把地址换掉，这样就算恶意用户提前拿到了地址，但是拿到的也是一个不可用的地址。

（3）       服务端可以通过管理后台来随时修改接口的地址。




## *缓存服务|页面优化

提供两类四种的缓存；

两类： 基于 Redis 的分布式缓存，基于 Encache 的本地缓存；

四种： 页面、URL、对象、数据结构缓存；



**&& 页面缓存**

在搭建集群的情况下，可以放在代理服务器中；

通过模板解析获取到对应的动态数据，之后通过视图解析器将动态数据注入进去，最终形成对应的 HTML 页面；

```java
@RequestMapping(value="to_list", produces="text/html;charset=utf-8")
```


**&& URL 缓存**

对于指定的一个商品页面，将其熏染放入到缓存中；


**&& 对象缓存**

（1） 缓存订单

根据 (userId, goodsId) 类似的动态规划表，缓存到 Redis 中，从而避免多次查询订单；

同时根据业务需要设置对应的有效期，即保证在秒杀期间可以访问该订单对象；

在执行秒杀时对用户是否秒杀过进行判断，以及在消息队列中进行是否秒杀过进行判断，都可以直接读取缓存，无需访问 DB；

① 秒杀中对于能否进行秒杀的判断：

```java
// ---- SeckillController ----
SeckillOrder order = iOrderService.selectSeckillOrderByUserIdAndGoodsId(user.getId(), goodsId);
if (order != null) {
	log.error("【秒杀】秒杀重复,userId:{},goodsId{}", user.getId(), goodsId);
	return ResultVO.error(ResultEnum.SECKILL_REPEATE);
}
```

② 队列中进行的逻辑判断：

```java
// ---- MQReceiver ----
SeckillOrder order = iOrderService.selectSeckillOrderByUserIdAndGoodsId(user.getId(), goodsId);
if (order != null) {
	log.error("【消息队列】重复的秒杀, userId: {}, goodsId: {}", user.getId(), goodsId);
	return;
}
```

③ 获取秒杀的结果：

```java
SeckillOrder order = iOrderService.selectSeckillOrderByUserIdAndGoodsId(userId, goodsId);
if (order != null) {
	return order.getOrderId();
} else if (checkGoodsIsOver(goodsId)) {
	return Const.SeckillOrderStatusEnum.OVER.getCode();
} else {
	return Const.SeckillOrderStatusEnum.WAIT_ON_QUEUE.getCode();
}
```



（2） 缓存用户信息

在修改的情况下，需要先更新 DB，后删除缓存的策略；

 Cache Aside Pattern :
 失效  : 先从 cache 取，没得到，则从 db 中取，成功后，放到 cache 中
 命中 : 从 cache 中取，取到后返回
 更新 : 先把数据放到 db, 成功后，让 cache 失效

 先更新 DB， 后删除缓存策略





非全量更新，只更新对应的字段即可；

```
SeckillUserKey:id:18852860000   		SeckillUser
```





删除策略：

更新数据库；

删除 id 缓存；

重新设置 token；





**&& 本地缓存**

可以在本地服务器添加 Encache 来防止缓存雪崩，在出现缓存雪崩后减轻一下 DB 的压力；

用此作为高可用的一种使用；



**&& 数据结构缓存**

（） 缓存某个商品当前是否还有库存

即该商品是否秒杀结束





（） 缓存商品库存

进行预减，从而控制住内存标记，进而减少并发打到下面取缓存、放入队列以及访问 DB 的请求；



（） 缓存验证码结果

仅仅为结果，在结果正确后，返回具有一定时间有效期的 URL；

```

```





（） 缓存动态的 URL

```
SeckillKey:seckillPath:18852860000:2			dsjfldsjklfsjdlfjs
```





（） 缓存限流键





（） 缓存页面的浏览量







（） Session 缓存

```
SeckillUserKey:token:6fe00b57e08c444199ddf49138ff1874       SeckillUser
```







## 页面优化

**&& 页面静态化**

对商品

避免网络中的流量，流量对应着资源(￥)；





（1） HTTP 对静态资源缓存的字段

Pragma
Expir
Cache-Control
（2） 配置静态资源

缓存的实现

```properties
spring.resources.add-mappings=true
spring.resources.cache-period=3600
spring.resources.chain.cache=true
spring.resources.chain.enabled=true
spring.resources.chain.gzipped=true
spring.resources.chain.html-application-cache=true
spring.resources.static-locations=classpath:/static/
```

（3） 处理订单详情|商品详情的页面静态化

```java
ResultVO<OrderDetailVO> info(SeckillUser user, @RequestParam("orderId") Long orderId)
ResultVO<GoodsDetailVO> detail(SeckillUser user, @PathVariable("goodsId") Long goodsId)
```


**&& 静态资源优化**

engine： 根据 Nginx 进行

者多个 CSS、JavaScript 文件的访问请求编程

webpack 进行


（） CDN 缓存

根据位置，访问最近的节点；

根据网络流量和各节点的连接进行控制；






## 消息队列异步服务

（1） 引入原因

我们不是把请求直接去访问DB，而是先把请求写到消息队列，做一个缓存，然后再去慢慢的更新数据库。这样做以后，前端用户的请求可能不会立即得到响应是成功还是失败，很可能得到的是一个排队中的返回值，这个时候，需要客户端再去服务端轮询，因为我们不能保证一定就秒杀成功了。当服务端出队，生成订单以后，把用户ID和商品ID写到缓存中，来应对客户端的轮询就可以了。

（2） 相关原理

对应的为问答项目中通过  Redis 的 List 实现的异步框架；

生产者不是将消息直接发布到队列上，而是放到 Exchange 上，之后他通过 Exchange 路由绑定到对饮的队列上；



**&& 消息队列异步的实现**

（1） 配置队列

绑定对应的队列，实现 RabbitMQ 的集成

```java
@Configuration
public class MQConfig {
	public static final String SECKILL_QUEUE = "Seckill.queue.1";
	@Bean
	public Queue queue() {
		return new Queue(SECKILL_QUEUE, true);
	}
}
```

**（2） 队列实体**

消息的传输对象

```java
class SeckillMessage {
	User user;
	long goodsId;
}
```



**（3） 消息发送者**

生产者将消息发送到 Exchange 上

① 将传入的 Object 序列化成 String

②  转化并发送到指定的队列



**（4）  消息接受者**

消费者

监听对应的队列名称



业务逻辑如下： 

① 获取对应的消息将其转换成对应的实体；

② 校验该实体的合法性；

③ 逻辑性校验： 

- 进行库存判断

- 进行是否已经秒杀过判断

④ 执行对应的逻辑

- 执行秒杀



**&& 秒杀订单逻辑**

只有在扣库存的成功的时候，才能够进行对应的订单生成逻辑；

在失败后，设置商品秒杀结束标记，用于优化查询的逻辑；

```java
@Transactional
public OrderInfo seckill(SeckillUser user, SeckillGoodsVO goods) {
	boolean isSuccess = iGoodsService.descStock(goods);
	if (isSuccess) {
		// create orderinfo by user, goodsVo; may error for unique index
		OrderInfo orderInfo = iOrderService.createOrder(user, goods);
		return orderInfo;
	} else {
		// stock>0 execute, fail mean have no stock
		setGoodsOver(goods.getId());
		return null;
	}
}
```



订单的创建

先插入到 OrderInfo 表中

```java
@Transactional
public OrderInfo createOrder(SeckillUser user, SeckillGoodsVO goods) {
	OrderInfo orderInfo = new OrderInfo();
	orderInfo.setUserId(user.getId());
	orderInfo.setGoodsId(goods.getId());
	orderInfo.setCreateDate(new Date());
	orderInfo.setDeliveryAddrId(1L);
	orderInfo.setGoodsCount(goods.getStockCount());
	orderInfo.setGoodsName(goods.getGoodsName());
	orderInfo.setGoodsPrice(goods.getSeckillPrice());
	orderInfo.setOrderChannel(0);
	orderInfo.setStatus(1);
	// SelectKey 返回注入到 orderInfo 的 id 中.
	orderMapper.insertOrderInfo(orderInfo);
	
	SeckillOrder seckillOrder = new SeckillOrder();
	seckillOrder.setGoodsId(goods.getId());
	seckillOrder.setOrderId(orderInfo.getId());      // have orderId, use mybatis to injected
	seckillOrder.setUserId(user.getId());
	// control to prevent oversold
	orderMapper.insertSeckillOrder(seckillOrder);
	// put into redis
	redisService.set(OrderKey.getSeckillOrderByUidGid, BasePrefix.getKey(user.getId(), goods.getId()), seckillOrder);
	return orderInfo;
}
```



**&& 调用(秒杀结果)**

**（1） Seckill 的核心逻辑**

根据


（2） 异步结果

对于商品秒杀完毕的，在 Redis 中设置对应商品是否结束，在查询时来判断是秒杀结束还是商品正在排队中；

① 秒杀成功，返回订单号；

② 秒杀已结束，可能仍然在排队中；

③ 排队中；



@Q: RabbitMQ 用来做什么的？ 为什么使用消息队列？

用于消息的异步处理，

在本项目中用来进行流量削锋，避免短时间内大量的请求压垮服务器。

同时实现了应用上的解耦，不依赖与具体的 API 直接调用；

将请求放入到队列中， Server 按照其处理能力从消息队列中订阅消息进行处理。

@Q: 消息队列有什么好处？


项目优化：

前端优化：

浏览器方面的优化，将多个 CSS, JS 请求压缩统一处理；

使用 CDN 加速访问；

通过反向代理服务器进行分流处理；



后端优化：

分布式缓存；

异步处理；

集群进行横向扩展；

代码上优化；



## 秒杀辅助

**&& 秒杀过程及结束后的辅助**

**（1） 内存标记**

通过 ConcurrentHashMap 来实现的

```java
Map<Long, Boolean> localOverMap = new ConcurrentHashMap<>();
```

**（2） Redis 预减库存**

```
GoodsKey:goodsStock:2      23
```


**（3） 用于异步查询使用的库存是否还有**

参数为 Gid

```
Seckill:goodsOver:2          true
```



**&& 秒杀前的辅助**

**（1） 验证码**

某个用户对于某件商品的验证码的答案

注： 每次都只有一个答案，在刷新过后，直接覆盖原来的旧的答案

```
SeckillKey：verifyCode:18852860000:2						2
```


**（2） 动态 URL**

用于接口限流使用的；

不限制是什么商品，时间有限；

一般通过异步调用，前端直接控制 ajax 回调使用；

```
SeckillKey:seckillPath   					dsklfksdfksd
```


**（3） 限流**

语义： 某个用户访问某个 URL 的限制；

```shell
AccessKey:access:/seckill/path:18852860007          1
```



## 全局功能

**（1） 日志访问记录**

记录用户访问接口 URL、IP 的时间，参数


**（2） 登录拦截**


**（3） 用户 Session 获取**



# 可能的问题

@Q: 为何要进行两次MD5加密？两次的理由是什么？如何实现？

HTTP在网络中已明文传输，彩虹表

第一次防止网络传输中被截取破解，第二次防止数据库被盗取使用彩虹表破解。

第一次：对SALT常量在进行char拆分，实现前后组装。

第二次：创建随机SALT，并入库




## 库存控制

将商品的库存预加载到缓存中；

进行秒杀时进行预减库存；

在消息队列中执行 秒杀服务失败时设置库存已经无；



## 项目介绍 & 架构的设计

& 介绍

对少量的商品以较低的价格在某个时间段内进行销售，数量有限，从而造成秒杀。

通过此网络营销手段，制造轰动效应，进行网站的推广，提高辨认度。


& 设计

针对高并发的设计

基础业务部分：考虑到秒杀项目基于原来的架构，为附属功能，且存在的时长较短。 需要与原来的业务进行解耦，防止该系统的奔溃影响到主业务的运行。

表设计上便是从原始的用户、商品、订单抽取出来的，类似收货地址、商品信息则复用原有的架构。


& 优化

前端设计： 使用 CDN 加速，本项目中仅仅是使用 Bootstrap 提供的免费 CDN 服务，实现加载 JS, CSS 文件不经过服务器； 反向代理，用同学的服务器将用户的请求转发到不同的服务器上；


后端设计上： 为了防止用户进行不断刷新造成不必要的负载，借助 Redis 构建了分布式缓存；

为了防止 DB 无法瞬时处理高并发，使用消息队列对流量进行削锋并异步处理；

为了避免单台服务器奔溃而搭建集群；

后台代码上，也对应增加异步处理。



& 流程

用户请求流程如下：

通过前台 JS 脚本异步请求指定商品的秒杀时间，根据时间显示当前的秒杀按钮的明暗


只允许第一个提交的订单被发送到订单子系统；


整体的架构图





分析可能的问题：

秒杀并发太大，压垮服务器，主要业务无法运行；

不断刷新增加服务器压力；

网络带宽被运营商限制；

直接地址下单；


应对策略： 

将秒杀与正常的业务逻辑进行分开，从而作为独立应用，将其部署到其他服务器上，可设置独立域名处理；

对商品的详情页进行页面静态化；

租借带宽，并将秒杀商品页面缓存到 CDN 上；

URL 动态化










## 部署



```
mvn clean package -Dmaven.test.skip=true
mvn clean package -Dmaven.test.skip=true -Pdev

```







# 亮点

分布式 Session 中直接将对象注入到方法的参数上，便于之后的访问；



## *KEY 设计

通过模板方法模式实现；

提取出公共的过期时间、KEY 的前缀；

同时通过接口作为外部模块调用的合约，实现多态性；



在每个 KEY 中定义含有语义的 KEY，便于确定之后的参数；





便于监控优化：

每个前缀对应一项功能，从而方便地找到各种使用处，易于找出优化点；

```java
public interface KeyPrefix {
	
	int expireSeconds();
	
	String getPrefix();
}
```



```java
abstract class BasePrefix implements KeyPrefix{
	int expireSeconds;
	String prefix;
   
    public int expireSeconds() {
        return expireSeconds;
    }

    public String getPrefix() {
        String className = this.getClass().getSimpleName();
        return new StringBuffer().append(className).append(Const.SPLIT).append(prefix).append(Const.SPLIT).toString();
    }
}
```

带有语义的 KEY，便于操作，"代码及注释" 的编程风格；

```java
public static GoodsKey getSeckillGoodsStockByGid = new GoodsKey(Const.PERMANENT, "goodsStock" + Const.SPLIT);
```





## *一些问题



>  分析问题愿意 ⇒ 给出解决方案



是先减库存，之后进行订单



**@Q: 预减库存是否应该放在消息队列中？**

放在秒杀上的话可能造成不必要的预减，也没有办法进行恢复；



但实际的电商系统，比如像淘宝这种，由于库存管理也是一块复杂的业务，所以一般会独立出一个上下文，叫库存中心。然后这个库存中心独立于商品中心以及订单中心。当订单中心要求减库存时，只需要和库存中心进行交互即可。这样的方式，会让系统的职责更明确，商品中心不需要关心商品的库存了，只需要关注商品本身的属性信息即可。然后，本案例由于只是案例，所以没有独立出库存中心，即库存上下文。所以会议座位的库存管理放在会议管理上下文中。





是付款成功后进行真正的减库存，还是在订单生成后就见库存的逻辑；







**@Q: 问：秒杀的场景下，如何解决库存超卖问题？**

错误的回答：通过加分布式锁，通过数据库的乐观锁。。。

正确的回答：

秒杀场景下，并发会特别的大，有两种情况会导致库存卖超：

（1）一个用户同时发出了多个请求，如果库存足够，没加限制，用户就可以下多个订单。

（2）减库存的SQL上没有加库存数量的判断，并发的时候也会导致把库存减成负数。



对于（1）前端加验证码，防止合法用户快速点鼠标同时发出多个请求，在后端的miaosha_order表中，对user_id和goods_id加唯一索引，确保就算是刷接口一个用户对一个商品绝对不会生成两个订单。对于（2）需要在扣减库存的SQL上加上库存数量的判断，只有扣减库存成功才可以生成订单：
![图片描述](http://img.mukewang.com/5abc37b9000173f906870131.png)

解析：首先要分析问题的原因，然后再给出解决方案，不要上来就怎么怎么样，连问题都不清楚，谈何解决啊！









**@Q: 使用 HashMap 是否会出现线程安全问题？**

不会， HashMap 在初始化的时候就确定了对应的容量，不会进行容量的变更了，且初始的时候是单线程的，不存在之后进行扩容引发的死循环问题(JDK8 还有??)









若鱼老师你好,update miaosha_goods set stock_count = stock_count - 1 where goods_id = #{goodsId} and stock_count > 0这个sql语句是会让库存不会扣成负的,但是也不会导致事务回滚,也就是会造成订单和秒杀订单创建成功,但是库存却没有扣减这个bug
课程后面讲了，只有扣减库存成功才生成订单，所以，不存在你说的问题。前面不是跟你说了么，继续往后看。 





**@Q: Redis中的库存如何与DB中的库存保持一致？**

Redis中的数量不是库存，它的作  <u>用仅仅时候只是为了阻挡多余的请求透传到db，起到一个保护DB的作用</u>  。因为秒杀商品的数量是有限的，比如只有10个，让1万个请求去访问DB是没有意义的，因为最多只有10个请求会下单成功，剩余的9990个请求都是无效的，是可以不用去访问db而直接失败的。

因此，这是一个伪问题，我们是不需要保持一致的。





@Q: 1.   Redis预减成功，DB扣减库存失败怎么办？

两大类情况可导致redis预减成功而DB扣减失败：

（1）       如果一个用户发出了多个请求（不管何种手段），而这些所有的请求比所有其他用的请求都更快的到达了服务器，这个时候如果库存足够，就会出现redis预减多次，而只能下单成功一次（前提是：这个用户的多个请求比网站的其他用户的请求都更快的到达服务器，这在网络环境不可知的情况下，基本不可能）

（2）       还有就是在生成订单的过程中发生了不可预料的异常，也会导致redis扣减成功，而db扣减失败（如果是DB出现了异常，可能所有的订单都无法生成，但是只要存在redis预减，活动就可以正常结束）

因此，在初始化的时候，redis中的数量可以多于db的库存数量。



出现这种情况的后果是什么？

（1）       对用户而言，秒杀不中是正常现象，秒杀中才是意外，单个用户能否秒杀中本来就是小概率事件，出现这种情况对用户而言是没有任何影响的。

（2）       对商户而言，本来就是为了做活动拉流量拉人气的，卖不完还可以省一部分费用，但是活动还是正常参与了，也是没有任何影响

（3）       对网站而言，网站最重要的是用户体验，只要网站不崩，用户不骂娘，对网站也没有任何影响。

所以，卖不完是完全允许的，但是卖超是绝对不允许的！卖超的这部分钱商家是不会出的，需要网站自己来出。



可能存在卖不出去的情况，但绝对不允许超卖的情况，Redis 中缓存的库存与在 DB 中的库存本省就不是一致的，也无需一致的。





@Q: 秒杀的原理

在画架构图的时候说，保证每个组件都是由意义的，而非为了技术而技术的添加在上面的。

秒杀与其他业务最大的区别在于：

秒杀的瞬间，

（1）系统的并发量会非常的大

（2）并发量大的同时，网络的流量也会瞬间变大。



关于（2），最常用的办法就是做页面静态化，也就是常说的前后端分离，把静态页面直接缓存到用户的浏览器端，所需要的数据从服务端接口动态获取。这样会大大节省网络的流量，再加上CDN，一般不会有大问题。

关于（1），这里的核心问题就在于如何在大并发的情况下能保证DB能扛得住压力，因为大并发的瓶颈在于DB。如果说请求直接从前端透传到DB，显然，DB是无法承受几十万上百万甚至上千万的并发量的。所以，我们能做的只能是减少对DB的访问，前端发出了100万个请求，通过我们的处理，最终只有10个会访问DB，这样就可以了！针对秒杀这种场景，因为秒杀商品的数量是有限的，这种做法刚好适用！

那么具体是如何来减少DB的访问呢？

假如：某个商品可秒杀的数量是10，那么在秒杀活动开始之前，把商品的ID和数量加载到缓存，比如：Redis。服务端收到请求的时候，首先减一下Redis里面的数量，如果数量减到0随后的访问直接返回秒杀失败。也就是说，只有10个请求最终会去实际的请求DB。

当然，如果我们的商品数比较多，1万件商品参与秒杀，1万*10=10万个并发去请求DB，DB的压力还是会很大，这里就用到另一个非常重要的组件：消息队列。我们不是把请求直接去访问DB，而是先把请求写到消息队列，做一个缓存，然后再去慢慢的更新数据库。这样做以后，前端用户的请求可能不会立即得到响应是成功还是失败，很可能得到的是一个排队中的返回值，这个时候，需要客户端再去服务端轮询，因为我们不能保证一定就秒杀成功了。当服务端出队，生成订单以后，把用户ID和商品ID写到缓存中，来应对客户端的轮询就可以了。

这样处理以后，我们的应用是可以很简单的进行分布式横向扩展的，以应对更大的并发。









@Q: 为什么要单独维护一个秒杀结束的标志？

（1）       前面也提过，所有的秒杀相关的接口都要加上活动是否结束的标志，如果结束就直接返回了，包括轮询的接口，防止一直轮询没法结束。

（2）       管理后台也可以手动的更改这个标志，防止出现活动开始以后就没法结束这种意外的发生。









@Q: Redis挂掉怎么办？

线上的Redis服务器一般都会做replicate，也就是所谓的主从，使用哨兵机制实现监控和自动故障转移，当主机挂掉以后，从机自动升级成主机。

实际上不光是Redis，为了高可用，线上服务器的每一个节点都必须要防止单点故障，比如：nginx、mysql、rabbitmq等等。



@Q: RabbitMQ里面的消息如何才能不丢失？即使是服务器重启？

RabbitMQ不丢失消息要做到：

（1）       Exchage持久化

（2）       Queue持久化

（3）       发送消息的时候，设置MessageDeliveryMode为MessageDeliveryMode.PERSISTENT，这个也是默认的行为

（4）       消息手动确认


@Q: 1.   分布式Session，如何Cookie被禁用怎么办？

Cookie被禁用可以在url中传递参数：



@Q: 使用 Redis 实现分布式锁？

[地址](https://www.cnblogs.com/linjiqin/p/8003838.html)



![1554262967671](../assets/1554262967671.png)

![1554262972510](../assets/1554262972510.png)




## 其他

 Http Request  ⇒  Filter  ⇒  DispatcherServlet  ⇒  HandlerInterceptor  ⇒  Controller
    Controller
      ⇒  afterPropertiesSet(need I InitializingBean) ⇒  `@PostConstructor` ???  ⇒  `@InitBinder` 初始化参数绑定??  ⇒  HandlerMethodArgumentResolver ??
                  ⇒  `@Valid、ModelAndView、@RequestParam`可默认获取参数 ??  ⇒  参数绑定(bean 映射...)  ⇒  method body











