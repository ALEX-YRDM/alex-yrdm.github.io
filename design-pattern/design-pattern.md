# 设计模式

https://www.digitalocean.com/community/tutorials/java-design-patterns-example-tutorial

## 单例模式

- 单例模式限制了类的实例化,确保在JVM中只存在该类的一个实例
- 该类必须提供一个公共的访问方式来获取类的实例
- 一些必须采取的措施:
  - 构造方法用private修饰
  - private static修饰和类名相同属性是唯一的实例
  - public static方法返回类实例

### Eager Init (饿汉模式)

```java
public class EagerSingleton{
    private static EagerSingleton singleton = new EagerSingleton();
    
    private EagerSingleton(){}
    
    public static EagerSingleton getInstance(){
        return singleton;
    }
}
```

```java
//静态代码块创建实例,和饿汉相似
public class StaticBlockSingleton{
    private static StaticBlockSingleton singleton;
    
    static{
        try{
            singleton = new StaticBlockSingleton();
        }catch(Exception e){
            throw new RuntimeException("error");
        }
    }
    
    private StaticBlockSingleton(){}
    
    public static StaticBlockSingleton getInstance(){
        return singleton;
    }
}
```

### Lazy Init懒加载

只有在第一次使用时才会创建该类实例对象

```java
public class LazyInitSingleton{
    private static LazyInitSingleton singleton;
    
    private LazyInitSingleton(){}
    
    private static LazyInitSingleton getInstance(){
        if(singleton == null){
            singleton = new LazyInitSingleton();
        }
        return singleton;
    }
}
```

但是上面代码会出现线程安全问题, 多线程同时访问时可能会创建多个实例对象

### 线程安全类 (第一次尝试)

直接使用在获取实例方法上使用synchronized关键字,确保多线程环境下,在某一时刻只有一个线程能够访问该方法,其他线程则阻塞

```java
public class ThreadSafeSingleton{
    private static  ThreadSafeSingleton singleton;
    
    private ThreadSafeSingleton(){}
    
    private  static synchronized ThreadSafeSingleton getInstance(){
        if(singleton == null){
            singleton = new ThreadSafeSingleton();
        }
        return singleton;
    }
}
```

添加synchronized确实解决了线程安全问题,但是却牺牲了性能,同一时刻只有一个线程可以访问该方法

### 线程安全(第二次尝试)

使用两段检查锁,同时添加volatile关闭指令重排

```java
public class ThreadSafeSinglrton{
    private static volatile ThreadSafeSingleton singleton;
    
    private ThreadSafeSingleton(){}
    
    private static ThreadSafeSingleton getInstance(){
        if(instance == null ){
            synchronized(ThreadSafeSingleton.class){
                if(instance == null){
                    instance = new ThreadSafeSingleton();
                }
            }
        }
        return singleton;
    }
}
```

### Bill pugh (优雅)

```java
public class BillPughSingleton{
   
    private BillPughSingleton(){}
    
    private static class SingletonHelper{
        private static final BillPughSingleton singleton = new BillPughSingleton();
    }
    
    public static BillPughSingleton getInstance(){
        return SingleHelper.singleton;
    }
}
```

采用Java中静态内部类来实现,不仅线程安全,而且避免了同步带来的性能开销,并且是lazy加载

静态内部类SingletonHelper并不会在BillPughSingleton被加载时立即初始化,只有在第一次调用genInstance()方法时才会初始化态

内部类的初始化是由JVM来保证的，根据Java语言规范，类的静态成员变量的初始化在类加载时是线程安全的