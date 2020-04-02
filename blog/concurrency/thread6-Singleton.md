# 单例写法

```
package Singleton;

public class SingletonObjectV1 {

    //主动加载方式之一: 问题 : 静态的实例,如果使用,长期占用内存空间
    //不能懒加载
    private static final SingletonObjectV1 instance= new SingletonObjectV1();

    private SingletonObjectV1() {
    }

    public static SingletonObjectV1 getInstance () {
        return instance;
    }
}

package Singleton;

/**
 * 懒加载的多线程安全问题;
 */
public class SingletonObjectV2 {

    private static SingletonObjectV2 instance;

    private SingletonObjectV2() {
    }

    /**
     * 此方法不是线程安全的, 有可能两个线程会返回不同的instance
     * @return
     */
    public static SingletonObjectV2 getInstance () {
        if (instance == null)
             instance=new SingletonObjectV2();

        return instance;
    }
}


package Singleton;

/**
 * 懒加载的问题解决1
 */
public class SingletonObjectV3 {

    private static SingletonObjectV3 instance;

    private SingletonObjectV3() {
    }

    /**
     * 解决办法1: 加一个Class锁,可以解决
     *     引入新的问题性能问题: synchronized 会让程序串行化,
     *     而且多数情况下,只是做读操作,没必要每次都要拿锁等待(只有第一次才会创建)
     * @return
     */
    public synchronized static SingletonObjectV3 getInstance () {

        if (instance == null)
             instance=new SingletonObjectV3();

        return instance;
    }
}



package Singleton;

/**
 * 懒加载的问题解决2
 */
public class SingletonObjectV4 {

    private static SingletonObjectV4 instance;

    private SingletonObjectV4() {
        //todo something
        //耗时操作
    }

    /**
     * 解决办法2: 双重判断double check, 不用每次获取instance都要抢锁,
     * 第一次创建instance如果是多个线程抢锁,第一个拿到锁的创建,后面拿到锁了判断不为null,也不会执行
     * 后面获取实例就不需要锁了,相当于是只读操作了,解决了性能问题
     *
     *     有可能存在问题: 空指针正异常, new构造创建的实例还没有创建完整,确实不为null,
     *     这个时候另个线程请求返回的instance不是完整的instance,操作某些方法有可能空指针
     *     //编译优化 //运行优化, JVM执行的时候不一定是按照顺序执行的
     *
     * @return
     */
    public  static SingletonObjectV4 getInstance () {

        if (instance == null)
             synchronized (SingletonObjectV4.class) {
                 if (instance == null)
                 instance = new SingletonObjectV4();
             }
        return instance;
    }
}


package Singleton;

/**
 * 懒加载的问题解决2
 */
public class SingletonObjectV5 {

    private static volatile SingletonObjectV5 instance;

    private SingletonObjectV5() {
        //todo something
        //耗时操作
    }

    /**
     * 解决办法2: 双重判断double check, 不用每次获取instance都要抢锁,
     * 第一次创建instance如果是多个线程抢锁,第一个拿到锁的创建,后面拿到锁了判断不为null,也不会执行
     * 后面获取实例就不需要锁了,相当于是只读操作了,解决了性能问题
     *
     *     有可能存在问题: 空指针正异常, new构造创建的实例还没有创建完整,确实不为null,
     *     这个时候另个线程请求返回的instance不是完整的instance,操作某些方法有可能空指针
     *      //编译优化 运行优化, JVM执行的时候不一定是按照顺序执行的
     *      解决办法 :  instance实例用 volatile修饰,
     *              不能保证原子性,
     *              可以保证内存的可见性(多个线程看到的数据是一致的)
     *              有序性 - 保证读的时候,写程序已经全部完成;
     *      虽然可以解决,但是volatile这个也有性能问题;
     *
     *
     * @return
     */
    public  static SingletonObjectV5 getInstance () {

        if (instance == null)
             synchronized (SingletonObjectV5.class) {
                 if (instance == null)
                 instance = new SingletonObjectV5();
             }
        return instance;
    }
}



package Singleton;


/**
 *  不加锁解决办法:
 *      static被初始化一次. 而且懒加载,实现了单例模式
 */
public class SingletonObjectV6 {

    private SingletonObjectV6() {

    }

    public static SingletonObjectV6 getInstance() {
        return InstanceHolder.instance;
    }

    private static class InstanceHolder {
        private final static SingletonObjectV6 instance = new SingletonObjectV6();
    }
}



package Singleton;


/**
 *  枚举解决办法:
 *      枚举类的特性,枚举类线程安全,实例只会加载一次
 */
public class SingletonObjectV7 {

    private SingletonObjectV7() {

    }

    public  static SingletonObjectV7 getInstance() {
        return Singleton.INSTANCE.getInstance();
    }

    /**
     * 枚举类型,线程安全, 构造只会被加载一次
     */
    private enum Singleton {
        INSTANCE;

        private final SingletonObjectV7 instance;

        Singleton () {
            instance = new SingletonObjectV7();
        }

        public SingletonObjectV7 getInstance() {
            return instance;
        }
    }
}


```
