java反射与动态代理

1. 代理类和委托类通常实现相同的接口，通过代理类可以有效的控制对委托类的直接访问
2. java.long.reflect.InvocationHander:调用处理接口，用于集中处理在动态代理类上的方法调用
3. java.long.ClassLoader:类装载器，负责将类的字节码装载到JVM并为其定义类对象，然后该类才能被使用


代理机制及其特点
1. 通过实现InvocationHander 接口创建自己的调用处理器
2. 通过Proxy类指定ClassLoader对象和一组interface来创建动态代理类
3. 通过反射机制来获得动态代理类的构造函数，其唯一的参数类型是调用处理器接口类型
4. 通过构造函数创建动态代理类实例，构造时调用处理器对象作为参数传入