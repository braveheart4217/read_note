##Cglib Manual

###Enhancer
Let's start with the Enhancer class, the probably most used class of the cglib library. An enhancer allows the creation of Java proxies for non-interface types. The Enhancer can be compared with the Java standard library's Proxy class which was introduced in Java 1.3. The Enhancer dynamically creates a subclass of a given type but intercepts all method calls. Other than with the Proxy class, this works for both class and interface types. The following example and some of the examples after are based on this simple Java POJO: