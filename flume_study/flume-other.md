##拦截器
1. 设置在Source和Channel之间的组件；每个拦截器只能处理同一个Source接受到的事件（即与一个Source实例绑定）；
2. 唯一配置参数是：type参数

##Channel选择器
1. 决定Source接收的某个特定事件写入那些Channel的组件，告知Channel处理器将事件写入到每个Channel；
2. Flume中重复数据的情况之一：一个事件需要写入到多个Channel中，而每个事件写入一个Channel后会在写入下一个Channel前提交，所以当一个channelA写入失败后，可能存在一个ChannelB已经被写入了数据并提交了，所以在事务回滚时，会试图写入相同的事件，导致重复；
3. Channel选择器分为两种：replicating(复制选择器)和multiplexing(多路复用选择器)；参数type就是填写选择器的类型的；
4. Channel选择器的参数前缀为Source，如：agent1.source.source1.selector.type = ...;

###复制选择器replicating
1. 该选择器是将所有事件数据复制到所有Source所配置的channel中；不会对数据做任何处理或者流的分叉，仅仅是复制数据；
2. 有一个**optional属性**，该属性是指明哪些channel是可选的；“可选的”含义是：写入这些标记为可选的channel时，发生写入失败等错误时，不会抛出异常和回滚；

###多路复用选择器multiplexing
1. 通过把事件的报头的某个特定属性（某个特定报头）的值作为判断依据，设置其为不同值时将事件发送到不同的channel中；
2. 需要设定的属性
    ![配置表](img/选择器参数(1).jpg)
    ![配置表](img/选择器参数(2).jpg)
    >* 其中type一定要设置为multiplexing；
    >* header就是特定报头属性值
    >* mapping.<value>设置当header的属性A的值为value时，将数据发送给哪个channel；
    >* optional.<value>这里的value可以与mapping的value不一样；
3. 如果optional的value有不一样，则如果有报头的值匹配这个value之后，会发送给这个optional的channel以及