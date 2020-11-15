## 数据模型

**快速构造一个类**

使用构建器实现，按照需要快速将参数注入进去

使用配置实现，自动解析所定义的字符串成对应类的属性



**通用常量设计**

(一) 借助 Enum 实现

1、定义说明属性

(1) 说明属性

capation: 控制前台展示的内容

order: 控制前台展示的顺序

(2) 含义属性

orgType、moduleName、interfaceName



2、枚举分组

通过将几个特定的 Enum 赋予一个组属性，通过 `isXX()` 进行区分，对枚举进行一次抽象

```java
boolean isTake()
boolean isRollback()
boolean isRecyle()
...
```

(二) 接口和类静态参数实现

1、普通类型

2、集合类型





**层级数据模型设计**

1、DB 设计

(1) level 添加方式

code
varchar(30)
类别代码，非空字段，同一企业下不允许重复，新增时自动将上级代码加到用户输入的代码前面，如上级代码为01，用户新增下级时输入代码02，则保存到数据库代码为0102

level
varchar(30)
类别级别，非空字段，用户自定义，采用预定义类型

(2) x.y.z 方式

通过字符串指明从根节点到当前节点的路径



2、带有层级实体的方法封装：
`T getDirectParent`: 获取层级实体的直接上级
`List<T> getDirectChildXX`: 获取层级实体的直接下级
`List<T> getChildXXs`: 获取层级实体下的所有下级实体
`T getRootByChild`: 获取到根节点
`Map<T, List<T> getXXRelationByChild(List<T>)`: 根据子节点获取对应的父节点 
`Map<T, List<T>> getXXRelationByParent(List<T>)`: 根据父节点获取对应的子节点







## 实体封装映射

**针对项目重新封装了 PXX 相关实体**
对DB列名与Bean 属性名进行映射
针对标准实体添加企业主键


**导入结果实体封装**
结构设计：

- 汇总信息，可选 {字符串输出, 统计数量，...}
- 具体信息

```java
class ImportError<T> {
  T data;
  String errMsg;
}
class ImportResult {
  int successCount;
  int errorCount;
  String errorDownloadUrl;
}
```



**查询参数实体的封装**
`<Entity>Filter`： 在类中定义好传入的参数，通过 XXGreaterThan, XXEquals, XXLike 方便前台对于参数理解，也方便后台进行 SQL 的拼凑实现


系统中有两种实现，一种是 XXFilter 中定义的全部是 String，另一种 XXFilter 中定义的是对应的类型，如枚举类型
前一种方便，不需要对接收的参数进行转化，直接访问 DB
后一种方式可以预防无效的参数从而提前结束运行，单需要对结果进行两次转化

实体之间的转化
通过 rumba 实现，借助 ConverBuilder 良好工具的封装：
对基本的映射；
对参数不一致的映射；
对参数类型不一致的映射；



**P实体**
数据库辅助查询对象
定义与数据库对应的属性名称
与 Entity 的转换接口
处理 column 与程序中的实体不一致的问题
1、通用的常量字段
2、实体基本的字段
3、JaveBean 到数据库实体的映射
级联 Entity, VersionEntity, StandardEntity 中， uuid, version, ...., .... 属性
4、数据库实体 到 JavaBean的映射
(1) 基本的实体信息
(2) 继承的父类实体信息
(2) 内嵌的对象属性
5、jdbc 封装返回的查询结构 RowMapper





## 工具封装

封装考虑参数:

最长使用的类型，一般是 String 类型，对应基本类型封装....

通用 Object 或泛型扩展

通过的 List 集合扩展



封装考虑线程安全性:

使用 ThreadLocal 方式保证线程安全

使用 Lock 或是 Synchronized 保证线程安全

使用线程安全的类





针对框架工具封装:

针对业务需要工具封装:

针对





针对 Spring 工具封装:

使用静态的工具类，需要定义自身对象成为静态属性，借助初始化注入进参数

使用 Spring 封装的工具、组件





**集合操作**

保持原有顺序，对其中内容进行去哄





**应用功能封装**
Application 实现的接口：
MessageSource： 提供对于国际化信息的支持
EnvironmentCapable： 实现获取环境配置信息
ApplicationEventPublisher： 进行 Spring 事件的发布
HierarchicalBeanFactory： 层级的
确定扫描的 feign 声明式 api 的包位置
确定开启 Swagger 文档，以及对应 Swagger 的配置


**Aop 工具封装**
通过 JointPoint 获取方法、特定名称的方法、

参数类型

方法上的注解、方法上特定的注解


**反射相关工具封装**
1、BeanUtils
对 Spring 的 BeanUtils 进行二次封装：
控制异常
控制赋值时忽略源对象中的空属性

将集合容器中的元素按照元素中的某个属性映射成 Map：
对于存在相同属性值，直接覆盖
支持多重嵌套的属性
仅支持一个属性， 无法实现多个属性的组合

将集合容器中的元素按照元素的某个属性映射成 Map: 支持重复，重复的使用 List 保存
返回的类型为 `Map<K, List<V>>`
按照某个属性进行分组控制

将集合容器中的元素，按照该元素的某个属性收集结果成一个 Set 返回：
实现查询结果元素属性去重


获取某个对象中的某个属性：
支持多重嵌套

2、EntityFieldUtil
将一个类的实体和值映射成一个 Map

对某个实体，特定的属性集合取出对应的值，最后映射成一个 Map


不支持深层的映射，仅实体的成员属性
支持 boolean和其他类型的命名风格


**通用验证器封装**
通过 ThreadLocal 进行封
IConverter ：
增强的转换器

提供集合类型的转换

MockUtils：
测试工具

PatternContants:
封装对应的编译后的正则表达式

QpcStrUtils 中对于 qtyStr 的比较：
m + n， n 可以为小数

件数????

数量和规格：
@Q
divideAndRemainder, 商和余数
BigDecimal split = new BigDecimal(strs[1]);
split.setScale(4);

统计工具：
@Q:

TimeUtil：
JDK8 封装的日期工具
封装常见的日期格式化的形式

上传工具：
UploadHelper
将上传的文件转化成 TypeReference 定义的对象
Map<..>, List<..>

**Excel 解析写入工具**
1、上传的 MultipartFile 解析成特定类型的对象

> Case: `List<XXCreation>`

将 MultipartFile 解析成 TypeReference<T> 中的 T 类型数据

解析 Excel 中格式支持：
支持子标题，通过 `（）` 进行分隔，内部可通过 `;` 进一步拆分子标题
标题支持富文本，对特定标题显示红色，作为必填项    @Q: 是否校验该项对应的列值为空?

内容支持 boolean 类型的值，数字类型，特定的日期格式
支持多 Sheet

错误定位：
在已知 `readValue` 异常信息的情况下处理
取出按照 `'` 分隔后对应的值即为对应解析出现错位的列


实现：
最终字符拼接的结果为 `[{}, {}, ...]` 的数组
之后读取字符结果成为给定的 `TypeRefrence<T>`

2、基本的Excel工具
(1) 写入Excel
用于导入失败Excel文件的生成
(2) 根据 InputStream 读取
返回 `List<Map<String, String>>` 格式





## 规范

设计公共的方法参数:
正常收货
快速收货
诚信收货
生鲜收货





常用的标志:
FIXME: 有问题的
TODO: 待完成的  

  


完成提炼变量（119）所需的简单结构调整    
查看方法的调用位置  

  

可以向某个参数发起查询而获得另一个参数的值，那么就可以使用以查 询取代参数   
如果你发现自己正在从现有的数据结构 中抽出很多数据项，就可以考虑使用保持对象完整（319）手法，直接传入原来 的数据结构。    
如果有几项参数总是同时出现，可以用引入参数对象（140）将其 合并成一个对象    
如果某个参数被用作区分函数行为的标记（flag），可以使用 移除标记参数    



  

**提炼函数（Extract Function）**  







**提炼变量，引入解释性变量**      

给调试器和打印语句提供了便利的抓 手。  
如果这个名字只在当前的函 数中有意义，那么提炼变量是个不错的选择；但如果这个变量名在更宽的上下文 中也有意义，我就会考虑将其暴露出来，通常以函数的形式。  

```
class Order { 　constructor(aRecord) { 　　this._data = aRecord; 　}

　get quantity() {return this._data.quantity;} 　get itemPrice() {return this._data.itemPrice;}

　get price() { 　　return this.quantity * this.itemPrice 　　　Math.max(0, this.quantity - 500) * this.itemPrice * 0.05 + 　　　Math.min(this.quantity * this.itemPrice * 0.1, 100); 　} }
```

这些变量名所代表的概念，适用 于整个Order类，而不仅仅是“计算价格”的上下文。

```
class Order { 　constructor(aRecord) { 　　this._data = aRecord; 　} 　get quantity() {return this._data.quantity;} 　get itemPrice() {return this._data.itemPrice;}

　get price() { 　　return this.basePrice - this.quantityDiscount + this.shipping; 　} 　get basePrice() {return this.quantity * this.itemPrice;} 　get quantityDiscount() {return Math.max(0, this.quantity - 500) * this.itemPrice * 0.05;} 　get shipping() {return Math.min(this.basePrice * 0.1, 100);} }
```





**改变函数声明**

函数明晨:

有一个改进函数名字的好办法：先写一句注释描 述这个函数的用途，再把这句注释变成函数的名字。

内联函数、内联变量    

  

只用过一次的代码保持内联状态  
以它“做什么”来命名  
    
    
    
函数之中嵌套函数，可以访问内部定义的所有变量  

  

根据指定的读局部变量构造出一个新对象  
修改原来的局部变量，且之后需要使用该局部变量  
更改  



重构词汇表:  
refreshEntity  
refreshTotalInfo  





**信息展示(M)**

日志记录的内容，因为什么而插入的





**对象创建(M)**

Converter 方式，字节码增强的，Creation -> Entity，多参数相同的，类似 `BeanUtils`，性能更高，扩展性更高

根据虚拟的数据基础上添加额外的字段，虚拟的

根据公共的基础信息，不断 clone 填充差异性的字段

Builder，先通过 Builder 的构造器注入基本的参数，之后根据需要注入其他的参数





注: 根据基本参数、原始参数生成的，需要注意之前是否已经被修改过

枚举项过滤；



将来源的一个属性，转换成目标的多个属性

将来源的多个属性，转换成目标的单个属性



多次涉及到 `binCode`, `binUsage`, `containerBarcode`, `qpcStr` 字段



参数对象

常用的一组参数封装起来







参数注入: 

对 List, Map 初始化，方便进行值的插入







图片的版本控制



**Predicate 参数**

使用 `Predicate` 作为类的参数，提供多参数的形参

```java
public final class Retry<T> implements BusinessOperation<T> {
  // ..
  private final Predicate<Exception> test;
  
    @SafeVarargs
  public Retry(
      Predicate<Exception>... ignoreTests
  ) {
    // ..
    this.test = Arrays.stream(ignoreTests).reduce(Predicate::or).orElse(e -> false);
  }
}
```





**执行器**

指定线程执行，最大的等待终止时间

Int -> Runnable -> executor.exec...

```java
final var inventory = new Inventory(1000);
var executorService = Executors.newFixedThreadPool(3);
// fast to map and then execute for multi thread
IntStream.range(0, 3).<Runnable>mapToObj(i -> () -> {
  while (inventory.addItem(new Item())) {
    LOGGER.info("Adding another item");
  }
}).forEach(executorService::execute);

executorService.shutdown();
try {
  executorService.awaitTermination(5, TimeUnit.SECONDS);
} catch (InterruptedException e) {
  LOGGER.error("Error waiting for ExecutorService shutdown");
}
```

**log 打印**

```java
var countries = world.fetch();
countries.stream().map(country -> "\t" + country).forEach(LOGGER::info);
```



Map 的 `forEach` 方法

```java
properties.forEach((key, value) -> builder.append("[").append(key).append(" : ").append(value).append("]"));
```



获取 `LinkedHashMap` 第一个元素

```java
int maxFreq = freqs.entrySet().iterator().next().getValue();
```



获取 `List` 第一个元素

> 用于获取 List 中存放元素的属性相同的值

```java
list.iterator().next();
list.get(0);
```