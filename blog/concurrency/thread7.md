# volatile

```
package Volatile;

/**
 * volatile 关键字
 */
public class VolatileTest {

    private  static int INIT_VALUE = 0;
    private final static int MAX = 5;

    public static void main(String[] args) {
        //JVM优化, jvm觉得这个线程没有写操作,不需要到主内存拿数据,所以一直使用的是自己的cache中的数据
        new Thread( () -> {
            //如果没有volatile修饰, INIT_VALUE值是被复制了一份,在自己的线程cache中, CPU cache
            int localValue = INIT_VALUE;
            while (localValue < MAX) {
                if (localValue != INIT_VALUE) {
                    System.out.printf( "The Value updated to [%d] \n",INIT_VALUE );
                    localValue = INIT_VALUE;
                }
            }
        },"Read").start();

        new Thread(() -> {
            //如果没有volatile修饰, INIT_VALUE值是被复制了一份,在自己的线程cache中,
            // 因为有写操作,所以修改之后会刷会主内存,再把缓存cache更新
            int localValue = INIT_VALUE;
            while (INIT_VALUE < MAX ) {
                System.out.printf("Update the value to [%d] \n",++localValue);
                 INIT_VALUE=localValue;
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"update").start();
    }
}


package Volatile;

/**
 * volatile 关键字
 *    内存cache, 缓存不一致
 *
 * 解决办法:
 *   1. 给数据总线加锁
 *        总线(数据总线,地址总线,控制总线), cpu1加锁执行,其他cpu无法执行,cpu1执行完毕,刷回每个cpu的缓存;
 *        引入问题: 效率问题,CPU串行
 *   2. cpu高速缓存一致性协议
 *        Intel MESI协议, 保证缓存和主内存一直,每个CPU的执行cache数据是一致的
 *
 *    核心思想 :  1.当CPU写入数据的时候,如果发现该变量被共享(其他CPU中也存在该变量的副本),
 *               会发出一个信号,通知其他cpu的该变量的缓存无效, 通知过程加锁
 *               2.当其他cpu访问该变量的时候,重新主内存获取, 获取过程加锁,
 *
 */
public class VolatileTest2 {

    private static int INIT_VALUE = 0;
    private final static int MAX = 5;

    public static void main(String[] args) throws InterruptedException {
        //T1有写数据操作,所以会更新主内存中的值,读取也会从主内存读取数据
        new Thread( () -> {
            while (INIT_VALUE < MAX) {
                System.out.println( "T1->" + (++INIT_VALUE));
                try {
                    Thread.sleep(1_000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"T1").start();

        Thread.sleep(1_000);

        new Thread(() -> {
            while (INIT_VALUE < MAX ) {
                System.out.println("T2->"+ (++INIT_VALUE));
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"T2").start();
    }
}




package Volatile;

/**
 * 并发编程的重要概念
 * 1. 原子性
 *       一个或者多个操作,要不全部成功,或者全部失败,不能因为任何原因中途失败;类似数据库事务
 * 2. 可见性
 *    一个线程修改数据,所有线程可见数据变化,解决 缓存一致性问题,CPU缓存问题
 * 3. 有序性
 *    代码执行顺序,指令重排序,保证最终一致性,中间变量的创建和赋值可能顺序和写的代码顺序不一致
 *
 *
 * JVM的支持
 * 1. 原子性
 *  对于基本类型的变量读取和赋值是保证了原子性的, 要么都成功,要么都失败,这些操作不可中断
 *  a =10 ; 原子性
 *  b =a ;  不满足 1,read a ; 2 assign b;
 *  c++;    不满足 1. read a; 2 add ; 3 assign to c
 *  c=c+1; 不满足 1. read a; 2 add; 3 assign to c
 *
 * 2. 可见性
 *   Volatile关键字保证可见性, 被volatile修饰的关键字,线程执行到主存取数据,不是在自己线程的cache中
 *
 * 3. 有序性
 * happens-before relationship
 *  3.1 在单线程内,编写再前面的先执行
 *  3.2 unlock必须发生在lock控制后
 *  3.3 volatile修饰的变量,对该变量的写操作先用对该变量的读操作
 *  3.4 传递规则,操作A先于B,B先于C,那么肯定A先于C
 *  3.5 线程启动规则start方法肯定先于线程的执行run方法
 *  3.6 线程的中断规则,interrupt这个动作,必须发生在捕获该动作之前
 *  3.7 对象的销毁规则,初始化必须发生在finalize之前
 *  3.8 线程终结规则, 所有操作都发生在线程死亡之前
 *
 *
 *  Volatile关键字语义
 *  1. 对其他线程修改的可见性,修改时主内存数据改变,其他线程的缓存失效
 *  2. 禁止jvm重排序,保证顺序性
 *  3. 不能保证原子性
 *       System.out.println( "T1->" + (++INIT_VALUE))
 *       ++INIT_VALUE;
 * 1. get From main memory  INIT_VALUE->10;
 * 2. INIT_VALUE=10+1;
 * 3. INIT_VALUE=11;
 *
 *  在执行++的时候,有可能放弃Cpu执行权,还没来得及完成修改和刷回主内存,
 *
 *
 *  使用
 *  1. 状态量标记
 *     volatil boolean start = true;
 *   while(start) {
 *
 *   }
 *
 *   close () {
 *       start = false
 *   }
 * 2. 数据的一致性 单例模式
 */
public class VolatileTest3 {



}

```
