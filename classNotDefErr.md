There is a difference between `ClassNotFoundException` and `NoClassDefFoundError`

1. While first one denote that the class you are using is not in your classpath
2. Second one denotes that, the class you are using is in turn using another class that is not in your classpath


NoClassDefFoundError can occur for multiple reasons like

1. ClassNotFoundException, class not found for that referenced class irrespective of whether it is available at compile time or not(i.e base/child class).
2. Class file located, but Exception raised while initializing static variables
3. Class file located, Exception raised while initializing static blocks 