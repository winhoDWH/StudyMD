## bean

1. spring默认创建和获取bean的都是单例模式，即spring的IOC容器只维护该类实现的一个实例；
2. 可以在<bean>标签中加入scope="prototype"使其变成原型模式，即每获取一次就创建一个新的实例

## bean的自动装配

1. spring会在上下文中自动寻找并自动给bean装配属性
2. 不需要在配置中写具体使用哪个实例填入成员变量中；在xml中配置<bean>标签的autowire属性
   * byName：自动在容器上下文查找，和setXXX对应名字的beanId；
   * byType：自动在容器上下文查找，和对象属性类型相同的bean；（在有两个类型相同的bean时会报错）
3. 注解实现：
   * 前提：需要在xml中加入一些配置；==<context:annotation-config/>==
   * `@Autowired`标签可以在属性上使用，也可以在set方法上使用；使用了改标签后，可以不需要写set方法；该标签通过byType来匹配
   * @Nullable：标记该属性可以为null；
   * @Autowired标签有属性：required=false，此时才允许为空，否则会报错；
   * @Qualifier(value="name")：可从IOC容器中获取同类型不同名的实例；
   * @Resource标签，java的原生注解，先通过name寻找实例，找不到再通过type寻找；可以通过name标签属性指定实例名；
   * @resource和@autowired两个注解先类型后名称找？



## 注解开发

1. 类的创建与管理：<context:componect-scan/>使用`@Component注解`与<context:annotation-config/>
   * @service、@controller、@repository、@component功能一致

2. 属性注入：
   * @Value
3. 作用域 @Scope，配置IOC容器如何管理bean