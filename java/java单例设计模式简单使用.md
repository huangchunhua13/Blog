单例设计模式作为23种设计模式之一，也是我们平常开发中使用的最频繁的一种设计模式，这里对它的使用做一个简单的记录。
单例设计模式的写法主要可以分为两种：饿汉式和懒汉式
- 饿汉式的写法
```
public class Singleton {
    private static Singleton instance = new Singleton();

    private Singleton(){}

    public static Singleton getInstance() {
        return instance;
    }
}
```
- 懒汉式的写法
```
public class Singleton {
    private static Singleton instance;

    private Singleton(){}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                instance = new Singleton();
            }
        }
        return instance;
    }
}
```

一般情况下，以懒汉式的写法使用居多
