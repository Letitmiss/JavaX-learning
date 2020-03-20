```
package com.cong.concurrency;

import javax.sound.midi.Soundbank;
import java.util.PrimitiveIterator;

public class Concurrency05 {

    //方法区
    private  int i = 0;

    //引用地址存储到方法区
    private byte[] bytes = new byte[1024];

    private static int counter = 0 ;

    //JVM will creat thread main
    public static void main(String[] args) {
        //create a JVM stack
     /*   try {
            add(0);
        }catch (Error e) {
            e.printStackTrace();
            // StackOverflowError,死循环栈溢出异常,
            System.out.println(counter);
        }*/

        /**
         *  Thread(ThreadGroup group, Runnable target, String name, long stackSize)
         *
         *   指定了一个较高的值stackSize参数可以允许抛出一个前一个线程来实现更大的递归深度StackOverflowError
         *   main函数的stackSize是JVM创建的, 需要自己创建一个线程测试
         * 分配一个新的 Thread对象，以便它具有 target作为其运行对象，将指定的 name正如其名，以及属于该线程组由称作 group ，并具有指定的 堆栈大小
         */
         new Thread(null,new Runnable() {
             @Override
             public void run() {
                 try {
                     add(1);
                 }catch (Error e) {
                     e.printStackTrace();
                     System.out.println(counter);
                 }
             }

             private void add(int i) {
                 ++counter;
                 add(i+1);
             }
         },"Test",1 << 24).start();

        /**
         * stack参数代表线程栈的大小, 如果没有指定的默认是0, 0 表示该平台会忽略该参数, 该参数被JNI去加载 -Xxxm10M
         *
         * http://www.matools.com/api/java8
         */

    }


}

```
package com.cong.concurrency;

public class Concurrency06 {

    private static int counter = 1;

    public static void main(String[] args) {

        try {
            for (int i = 0; i < Integer.MAX_VALUE ; i++) {
                counter++;
                new Thread( () -> {
                    while (true) {
                        try {
                            Thread.sleep(1);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }).start();
            }
        }catch (Error error) {
            error.printStackTrace();
            System.out.println(counter);
        }
    }
}


```
package com.cong.concurrency;

import javax.sound.midi.Soundbank;

/**
 * Thread API介绍
 * 1. 构造函数,细节 threadgroup  stacksize 线程与线程组,线程与栈的关系
 * 2. 静态方法和非静态方法
 * 3.
 */
public class Concurrency03 {


    public static void main(String[] args) {

        /**
         * 构造函数
         * 1. Thread() 创建新的对象
         * 创建线程对象,默认线程名, Thread-0 开始计数 , 需要重写run方法,否则什么都不执行
         */
        Thread thread1 = new Thread(){
            @Override
            public void run() {
                System.out.println("Thread的构造API");
            }
        } ;
        thread1.start();


        /**
         * 构造函数
         * 2. Thread(Runnbale runnable) 创建新的对象
         *  构造的时候后如果没有传入runnable或者没有复写, run方法, 改Thread不会调用任何东西
         *  如果传递了runnable接口的实例,或者复写了Thread的run方法, 则会执行该方法的逻辑单元
         */
        Thread thread2 = new Thread( () -> {
            System.out.println("构造API实现runnable的接口");
        } ) ;
        thread2.start();

        /**
         * 构造函数
         * Thread(String name)
         */
        Thread thread3= new Thread("myThread");
        thread3.start();
        System.out.println(thread3.getName());

        /**
         * 构造函数
         * Thread(Runnable target, String name)
         */
        new Thread( () -> {
            System.out.println(Thread.currentThread().getName() + " start ....");
        }, "RunnableThread").start();


    }
}



```
package com.cong.concurrency;

import javax.sound.midi.Soundbank;
import java.util.Arrays;

public class Concurrency04 {

    public static void main(String[] args) {

        /**
         * 构造方法:
         *  Thread(ThreadGroup group, Runnable target)
         *  如果不给ThreadGroup默认是就是当前线程的爹的threadGroup
         *
         *  如果创建时未传入threadgoroup.默认以父线程的threadgroup作为创建线程的threadgroup,
         *  此时子线程和父线程在同一个threadgroup
         */
        Thread thread = new Thread("Test") {

            @Override
            public void run () {
                try {
                    Thread.sleep(1000*10L);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };
        thread.start();
        System.out.println(thread.getThreadGroup().getName());

        System.out.println(Thread.currentThread().getThreadGroup());
        System.out.println(Thread.currentThread().getThreadGroup().getName());

        ThreadGroup threadGroup = Thread.currentThread().getThreadGroup();
        int i = threadGroup.activeCount();
        System.out.println(i);

        Thread[] threads = new Thread[threadGroup.activeCount()];
        threadGroup.enumerate(threads);

        Arrays.asList(threads).forEach(System.out::println);




        /***
         * Thread(ThreadGroup group, Runnable target)
         分配一个新的 Thread对象。
         Thread(ThreadGroup group, Runnable target, String name)
         分配一个新的 Thread对象，使其具有 target作为其运行对象，具有指定的 name作为其名称，属于 group引用的线程组。
         Thread(ThreadGroup group, Runnable target, String name, long stackSize)
         分配一个新的 Thread对象，以便它具有 target作为其运行对象，将指定的 name正如其名，以及属于该线程组由称作 group ，并具有指定的 堆栈大小 。
         Thread(ThreadGroup group, String name)
         分配一个新的 Thread对象。

         栈内存与堆内存

         栈内存: 基本类型  引用类型(4byte)
         堆内存: 真实的存储的数据
         方法区域 : 多个线程共享
         虚拟机栈: 每次执行某个方法的内存, 内部局部变量等-线程独立的 : 局表变量表,操作栈, 动态链接 方法出口
         本地方法区: JDK的发操作
         */


    }
}

```
package daemon;

/**
 * 查看线程的状态
 * http://www.matools.com/api/java8
 */
public class DaemonThread {

    public static void main(String[] args) {

       Thread thread  = new Thread() {

           @Override
            public void run  (){
               try {

                   Thread t11 = new Thread(() -> {
                       try {
                           System.out.println(Thread.currentThread().getName() + "running");
                           Thread.sleep(1000 * 20L);
                           System.out.println(Thread.currentThread().getName() + "done");
                       } catch (InterruptedException e) {
                           e.printStackTrace();
                       }
                   }, "1-1");
                   //t11.setDaemon(true);
                   t11.start();

                   System.out.println(Thread.currentThread().getName()+"running");
                   Thread.sleep(1000*10L);

                   System.out.println(Thread.currentThread().getName()+"done");
               } catch (InterruptedException e) {
                   e.printStackTrace();
               }
           }
        }; //new

        //设置一个守护线程 :  守护线程是非守护线程的保姆
        //thread.setDaemon(true);

        //当jvm中的非守护线程结束的时候, 守护线程也终止结束,JVM退出, 如果jvm中有非守护线程,jvm就不能退出

        /**
         * 守护线程,代替非守护线程做事, 比如建立一个连接就立马建立一个守护线程去监听心跳, 如果连接断开的时候, 守护线程会自己退出, 辅助类的东西;
         * 如果建立的非守护线程,主线程结束了, 监听心跳的线程还会一直存在执行jvm不会退出
         */

        thread.start(); // runnable

        //runnable-->running --> dead --> block
        try {
            Thread.sleep(1000*5L);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + "stop");

    }
}

/**
 * 1. main线程已经提前执行结束了,为什么子线程还在执行, 并没有退出?
 *  因为main和thread属于同一个threadgroup,jvm看grou中还有active的线程,所以没有退出执行
 */


/**
 * 2. 守护线1中,创建一个守护线程2, 如果守护线程1结束, 守护线程2是否结束?
 *    
 *
 * 3. 守护线程中创建一个非守护线程, 守护线程结束, 非守护线程能否结束退出,jvm退出?
 *
 */
```

package threadapi;

import java.util.Optional;

public class ThreadAPI {

    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            Optional.of("Hello").ifPresent(System.out::println);
            try {
                Thread.sleep(100_000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"t1");

        t1.start();

        Optional.of(t1.getName()).ifPresent(System.out::println);
        Optional.of(t1.getId()).ifPresent(System.out::println);
        Optional.of(t1.getPriority()).ifPresent(System.out::println);
        //设置优先级,但是不一定按照设置的优先级别执行,优先级高获得cpu的执行权而已,开发不建议设置这个
    }
}

```

```
