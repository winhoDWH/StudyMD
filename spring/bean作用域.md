## bean

1. spring默认创建和获取bean的都是单例模式，即spring的IOC容器只维护该类实现的一个实例；
2. 可以在<bean>标签中加入scope="prototype"使其变成原型模式，即每获取一次就创建一个新的实例

## bean的自动装配

1. spring会在上下文中自动寻找并自动给bean装配属性