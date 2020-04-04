## Spring如何简化开发

- 基于POJO的轻量级和最小侵入式编程：不会因为Spring的API弄乱应用代码
- 通过依赖注入和面向接口实现松耦合
- 通过AOP实现声明式编程和减少样板式代码

## 依赖注入方式

- 构造方法注入
- setter注入
- 接口注入：强制被注入对象实现不必要的接口，带有侵入性，不提倡

## SpringBean作用域

- singleton : 唯一 bean 实例，Spring 中的 bean 默认都是单例的。

  >  IOC实例化时就创建

- prototype（原型） : 每次请求都会创建一个新的 bean 实例。

  > getBean时才会创建

- request : 每一次HTTP请求都会产生一个新的bean，该bean仅在当前HTTP request内有效。

- session : 每一次HTTP请求都会产生一个新的 bean，该bean仅在当前 HTTP session 内有效。

- global-session： 全局session作用域，仅仅在基于portlet的web应用中才有意义，Spring5已经没有了

## Spring事务传播行为

当事务方法被另一个事务方法调用时，必须通过Propagation指定使用原来的事务还是开启新的事务

- **REQUIRED**（默认值）：在当前事务运行，没有则开启一个事务。
- **REQUIRES_NEW**：在新开启的事务中运行，把当前事务挂起。
- SUPPORTS：在当前事务运行，否则以非事务方式执行。
- MANDATORY（强制）：当前没有事务，就抛出异常。
- NOT_SUPPORTED：以非事务方式执行，存在事务就把事务挂起。
- NEVER：以非事务方式执行，如果当前存在事务，则抛出异常。

## Spring事务隔离级别

通过Isolation指定，包括四种隔离级别和DEFAULT（数据库默认隔离级别），Oracle只支持已提交读和串行化

## Spring乱码问题

- post请求：web.xml配置过滤器，参数可查看源码`CharactorEncodingFilter`
- get请求：tomcat的server.xml中Connector添加参数URIEncoding="utf-8"（tomcat8默认已解决）

## SpringMVC工作流程

![SpringMVC运行原理](E:\Note\images\49790288-1585985903929.jpg)

- 客户端（浏览器）发送请求，直接请求到 DispatcherServlet。
- DispatcherServlet 根据请求信息调用 HandlerMapping，解析请求对应的 Handler。
- 解析到对应的 Handler（也就是我们平常说的 Controller 控制器）后，开始由 HandlerAdapter 适配器处理。
- HandlerAdapter 会根据 Handler 来调用真正的处理器（Controller里的具体方法）处理请求，并处理相应的业务逻辑。
- 处理器处理完业务后，会返回一个 ModelAndView 对象，Model 是返回的数据对象，View 是个逻辑上的 View。
- ViewResolver 会根据逻辑 View 查找实际的 View。
- DispaterServlet 把返回的 Model 传给 View（视图渲染）。
- 把 View 返回给请求者（浏览器）

