## 单例模式

- 单例模式限制了类的实例化,确保在JVM中只存在该类的一个实例
- 该类必须提供一个公共的访问方式来获取类的实例
- 一些必须采取的措施:
  - 构造方法用private修饰
  - private static修饰和类名相同属性是唯一的实例
  - public static方法返回类实例

### Eager Init (饿汉模式, 直接new和静态代码块)

```java
public class EagerSingleton {
    /**
     * 饿汉单例
     */
    //实例限定为private static final
    private static final EagerSingleton instance = new EagerSingleton();
    //构造函数为private 禁止外部new
    private EagerSingleton(){}
    //public static获取实例
    public static EagerSingleton getInstance(){
        return instance;
    }
}
```

```java
public class StaticBlockSingleton {
    /**
     * 静态代码块初始化，和饿汉一样
     */
    private static final StaticBlockSingleton instance;

    static{
        try{
            instance = new StaticBlockSingleton();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    private StaticBlockSingleton(){}

    public static StaticBlockSingleton getInstance(){
        return instance;
    }
}
```

### Lazy Init懒加载

只有在第一次使用时才会创建该类实例对象

```java
public class LazySingleton {

    private static  LazySingleton instance;

    private LazySingleton(){}

    public static LazySingleton getInstance(){
        if (instance==null){
            instance = new LazySingleton();
        }
        return instance;
    }

}
```

但是上面代码会出现线程安全问题, 多线程同时访问时可能会创建多个实例对象

### 线程安全类 (第一次尝试)

直接使用在获取实例方法上使用synchronized关键字,确保多线程环境下,在某一时刻只有一个线程能够访问该方法,其他线程则阻塞

```java
public class ThreadSafeSingleton {
    /*
     * 直接使用synchronized加在方法上，使得所有线程只能有一个在执行getInatance方法
     * 性能较差
     */
    private static ThreadSafeSingleton instance;

    private ThreadSafeSingleton(){}

    public static synchronized ThreadSafeSingleton getInstance(){
        if (instance==null){
            instance = new ThreadSafeSingleton();
        }
        return instance;
    }
}
```

添加synchronized确实解决了线程安全问题,但是却牺牲了性能,同一时刻只有一个线程可以访问该方法

### 线程安全(第二次尝试)

使用两段检查锁,同时添加volatile关闭指令重排

```java
public class DoubleCheckThreadSafeSingleton {
    /**
     * 必须使用volatile关闭指令重排
     * 
     * 两阶段检查：
     * 第一段检查：如果实例已经创建，可以极速返回
     * 如果没有创建：假设有2个线程AB进行同步代码块：
     * A拿到了锁，进入内部，发现instance仍为null，即创建并返回
     * 随后B拿到锁，进入内部，发现instance不为null，返回已有的实例，如果没有第二段检查，可能会创建多个instance
     * 
     * 关闭指令重排的原因是：
     * instance = new DoubleCheckSingleton() 并不是原子操作
     * 分为3步：
     * 第一步：分配内存空间
     * 第二部：调用构造函数，初始化对象
     * 第三部：将instance指向分配的内存地址
     * 在开启指令重排时，有可能将3移动到2前面执行
     * 即先将instance指向第一步分配的内存地址，再调用构造函数初始化对象
     */
    private static volatile DoubleCheckThreadSafeSingleton instance;

    private DoubleCheckThreadSafeSingleton(){}

    public static DoubleCheckThreadSafeSingleton getInstance(){
        if(instance == null){
            synchronized(DoubleCheckThreadSafeSingleton.class){
                if(instance == null){
                    instance = new DoubleCheckThreadSafeSingleton();
                }
            }
        }
        return instance;
    }
}
```

### Bill pugh (优雅)

```java
public class BillPughSingleton {
    /**
     * 使用java虚拟机类加载机制实现线程安全
     * 实现了：
     * 懒加载
     * 线程安全
     * 高性能
     */
    private BillPughSingleton(){}

    /**
     * jvm的类加载是懒惰的 只要没有主动访问SingletonHelper，JVM就不会加载和初始化它
     */
    private static class SingletonHelper{
        // 属于该类的静态初始化操作  JVM保证类的加载是线程安全的
        private static final BillPughSingleton instance = new BillPughSingleton();
    }

    public static BillPughSingleton getInstance(){
        return SingletonHelper.instance;
    }
}
```

采用Java中静态内部类来实现,不仅线程安全,而且避免了同步带来的性能开销,并且是lazy加载

静态内部类SingletonHelper并不会在BillPughSingleton被加载时立即初始化,只有在第一次调用genInstance()方法时才会初始化态

内部类的初始化是由JVM来保证的，根据Java语言规范，类的静态成员变量的初始化在类加载时是线程安全的

### 破坏单例模式

```java
public class DestorySingleton {
  
  /**
     * 使用反射破坏单例模式
     */
    @Test
    public void testBillPughSingleton(){
        /**
         *  top.stackpop.singleton.BillPughSingleton@16b4a017
            top.stackpop.singleton.BillPughSingleton@8807e25
            false
         */
        BillPughSingleton instance = BillPughSingleton.getInstance();
        BillPughSingleton instance2 = null;
        try{
            @SuppressWarnings("rawtypes")
            Constructor[] constructors = BillPughSingleton.class.getDeclaredConstructors();
            for(var constructor : constructors){
                constructor.setAccessible(true);
                instance2 = (BillPughSingleton)constructor.newInstance();
                break;
            }
        }catch(Exception e){
            throw new RuntimeException(e);
        }
        System.out.println(instance);
        System.out.println(instance2);
        System.out.println(instance == instance2);
    }
  
  	public void testDoubleCheckThreadSafeSingleton(){
        /**
         * 输出结果：
         * top.stackpop.singleton.DoubleCheckThreadSafeSingleton@3c679bde
            top.stackpop.singleton.DoubleCheckThreadSafeSingleton@16b4a017
            false
         */
        DoubleCheckThreadSafeSingleton instance = DoubleCheckThreadSafeSingleton.getInstance();
        DoubleCheckThreadSafeSingleton instance2 = null;
        try{
            @SuppressWarnings("rawtypes")
            Constructor[] constructors = DoubleCheckThreadSafeSingleton.class.getDeclaredConstructors();
            for(var constructor : constructors){
                constructor.setAccessible(true);
                instance2 = (DoubleCheckThreadSafeSingleton)constructor.newInstance();
                break;
            }
        }catch(Exception e){
            throw new RuntimeException(e);
        }
        
        System.out.println(instance);
        System.out.println(instance2);
        System.out.println(instance == instance2);
    }
}
```
上面6种都可以被反射破坏,因为反射可以访问私有构造方法

### 序列化和反序列化导致创建多个实例
```java
import java.io.Serializable;

public class SerializationSingleton implements Serializable{

    private static final long serialVersionUID = -1L;
    private SerializationSingleton(){}

    private static class SingletonHelper{
        private static final SerializationSingleton instance = new SerializationSingleton();
    }

    public static SerializationSingleton getInstance(){
        return SingletonHelper.instance;
    }
    
    /**
     *如果不重写该方法，导致反序列化时创建不同对象
     */
    protected Object readResolve(){
        return getInstance();
    }
}


		@Test
    public void testSerializationSingleton() throws FileNotFoundException, IOException, ClassNotFoundException{
        SerializationSingleton instance = SerializationSingleton.getInstance();
        ObjectOutput out = new ObjectOutputStream(new FileOutputStream("filename.ser"));
        out.writeObject(instance);
        ObjectInput in = new ObjectInputStream(new FileInputStream("filename.ser"));
        SerializationSingleton instance2 = (SerializationSingleton)in.readObject();
        in.close();
        System.out.println(instance.hashCode());
        System.out.println(instance2.hashCode());
        System.out.println(instance==instance2);
    }

		@Test
    public void destorySerializationSingleton() throws InstantiationException, IllegalAccessException, IllegalArgumentException, InvocationTargetException{
      /*
      反射同样可以破坏单例
      428746855
      317983781
      false
      */
        SerializationSingleton instance = SerializationSingleton.getInstance();
        SerializationSingleton instance2 = null;
        Constructor[] constructors = SerializationSingleton.class.getDeclaredConstructors();
        for(var c: constructors){
            c.setAccessible(true);
            instance2 = (SerializationSingleton) c.newInstance();
            break;
        }
        System.out.println(instance.hashCode());
        System.out.println(instance2.hashCode());
        System.out.println(instance==instance2);
    }
```

### 枚举单例
```java
public enum EnumSingleton{
    /**
     * 枚举不支持懒加载
     */
    INSTANCE;

    public static EnumSingleton getInstance(){
        return INSTANCE;
    }
}

		//下面测试代码，执行报错，枚举类不会被反射破坏
    @Test
    public void destoryEnumSingleton() throws InstantiationException, IllegalAccessException, IllegalArgumentException, InvocationTargetException{
        // EnumSingleton instance = EnumSingleton.INSTANCE;
        // EnumSingleton instance2 = null;
        // Constructor[] constructors = EnumSingleton.class.getDeclaredConstructors();
        // for(var c: constructors){
        //     c.setAccessible(true);
        //     instance2 = (EnumSingleton) c.newInstance();
        //     break;
        // }
        // System.out.println(instance.hashCode());
        // System.out.println(instance2.hashCode());
        // System.out.println(instance==instance2);
    }
```
枚举单例是线程安全的，且不会被反射和序列化破坏,但不支持懒加载