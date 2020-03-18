## Spring如何简化开发

- 基于POJO的轻量级和最小侵入式编程：不会因为Spring的API弄乱应用代码
- 通过依赖注入和面向接口实现松耦合
- 

## 依赖注入方式

- 构造方法注入
- setter注入
- 接口注入：强制被注入对象实现不必要的接口，带有侵入性，不提倡

## SpringBean生命周期

- singleton : 唯一 bean 实例，Spring 中的 bean 默认都是单例的。
- prototype : 每次请求都会创建一个新的 bean 实例。
- request : 每一次HTTP请求都会产生一个新的bean，该bean仅在当前HTTP request内有效。
- session : 每一次HTTP请求都会产生一个新的 bean，该bean仅在当前 HTTP session 内有效。
- global-session： 全局session作用域，仅仅在基于portlet的web应用中才有意义，Spring5已经没有了