前置：Before
后置：After
返回：AfterReturning
异常：AfterThrowing
环绕：Around

**切面类要有两个注解：@Aspect与@Component**，一个标记是切面类一个标记该类为bean

执行顺序：
around中前置的方法会比before要先执行；
around中后置方法会比after先执行；
after会比AfterReturning先执行；
around前置->before->around后置->after->afterReturn；

AspectJ 切入表达式的语法部分解释
https://www.iteye.com/blog/jinnianshilongnian-1420691
