# thread


## 构造
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
## 线程组构造方法
```
package com.cong.concurrency;

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
        System.out.println( "新建线程的Group :"+ thread.getThreadGroup().getName());

        System.out.println("main线程的Group : "+Thread.currentThread().getThreadGroup().getName());

        ThreadGroup threadGroup = Thread.currentThread().getThreadGroup();
        int i = threadGroup.activeCount();
        System.out.println(i);
        //3 说明还有一个其他线程

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

## Thread(ThreadGroup group, Runnable target, String name, long stackSize) 构造
```
package com.cong.concurrency;


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

# 守护线程

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
 *  因为main和thread属于同一个threadgroup,jvm看grou中还有active的线程,所以没有退出执行\
 *  
 * 2. 为子线程开启一个守护线程, 子线程结束, 守护线程退出, 这样可以通过非守护线程控制线程退出
 */

```
## 线程API

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
        Optional.of(t1.getId()).ifPresent(System.out::println); //11
        //启动虚拟就创建了10个线程了
        Optional.of(t1.getPriority()).ifPresent(System.out::println);
        //设置优先级,但是不一定按照设置的优先级别执行,优先级高获得cpu的执行权而已,开发不建议设置这个
    }
}

package threadapi;

import java.util.Optional;

public class ThreadAPI2 {
    public static void main(String[] args) {

        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 1000 ; i++) {
                Optional.of(Thread.currentThread().getName() + " index " + i).ifPresent(System.out::println);
            }
        });
        t1.setPriority(Thread.MAX_PRIORITY);

        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 1000 ; i++) {
                Optional.of(Thread.currentThread().getName() + " index " + i).ifPresent(System.out::println);
            }
        });
        t2.setPriority(Thread.NORM_PRIORITY);

        Thread t3 = new Thread(() -> {
            for (int i = 0; i < 1000 ; i++) {
                Optional.of(Thread.currentThread().getName() + " index " + i).ifPresent(System.out::println);
            }
        });
        t3.setPriority(Thread.MIN_PRIORITY);

        t1.start();
        t2.start();
        t3.start();
        
    }
}


package threadapi;

import javax.swing.text.html.Option;
import java.util.Optional;
import java.util.stream.IntStream;

public class ThreadAPI3 {

    public static void main(String[] args) {
        try {
            Thread t1 =  new Thread( () -> {
                IntStream.range(1,10000).forEach( i-> {

                    try {
                        System.out.println(Thread.currentThread().getName()+ "->" + i);
                        Thread.sleep(1000*1L);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                });
            });



            Thread t2 =  new Thread( () -> {
                IntStream.range(1,10000).forEach( i-> {
                    System.out.println(Thread.currentThread().getName()+ "->" + i);
                    try {
                        Thread.sleep(1000* 1L);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                });
            });

            t1.start();
            t1.join(); //当前线程执行结束,才会执行结束才会开始执行t2
            t2.start();

            //t1.join();  //t1和t2交替执行
            t2.join();  //当前线程执行结束,才会执行main线程的

            Optional.of("All of task finish done").ifPresent(System.out::println);
            IntStream.range(1,1000).forEach( i->
                    System.out.println(Thread.currentThread().getName()+ "->" + i));


        } catch (InterruptedException e) {
            e.printStackTrace();
        }


        //主线程和子线程会交替执行; 不确定会怎么执行;

        //期望t1线程执行结束再执行main线程 使用jion
    }
}

package threadapi;

import java.util.Optional;
import java.util.stream.IntStream;

public class ThreadAPI4 {

    public static void main(String[] args) {

        Thread t1 = new Thread( () -> {
            try {
                System.out.println("t1 is running");
                Thread.sleep(1000*10L);
                System.out.println("t1 is done");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        t1.start();
        try {
            //t1.join(); //等待直到结束才会继续向下执行
            //t1.join(1000*5); //等待5s,再去做main线程做的事情
            t1.join(5000,100); //5s 之后10 纳秒之后再做main方法的事情

        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        IntStream.range(1,1000).forEach(i->
                System.out.println(Thread.currentThread().getName()+ "->" + i));


        try {
            //自己等待自己结束,自己永远不结束
            //如果有一个线程的守护线程,非守护线程结束时,守护线程就会结束, 不希望守护线程退出
            //所以非守护线程需要自己判断自己是否结束,给你加入jion,这样非守护线程各种情况都不会退出
            //main在等自己结束, 自己做的事情,就是等自己结束, 一直等待下去没有结束了, 一直运行中
            Thread.currentThread().join();


        } catch (InterruptedException e) {
            e.printStackTrace();
        }

    }
}


//实例
package threadapi;
/**
 * 采集数据
 *  采集多态服务器的数据很多台服务器数据
 * 1. 一个线程采集一台数据
 * 2.所有线程都采集结束,表示采集结束
 *
 * */
public class ThreadJoin {

    public static void main(String[] args) {

        Thread machine1 = new Thread(new CaptureRunnable("machine1", 1000L),"machine1");
        Thread machine2 = new Thread(new CaptureRunnable("machine2", 10000L),"machine2");
        Thread machine3 = new Thread(new CaptureRunnable("machine3", 5000L),"machine3");

        machine2.start();
        machine1.start();
        machine3.start();

        //上面的任务还没有全部结束,下面的代码已经执行, 所以需要等上面的任务全部结束才向下执行
        //java提供丰富的线程包,不需要直接使用这个操作join,等待线程结束;
        try {
            machine1.join();
            machine2.join();
            machine3.join();

            System.out.println("All task is done, update database" + System.currentTimeMillis() );

        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}


class CaptureRunnable implements  Runnable {

    private String machineName;

    private long time;

    public CaptureRunnable(String machineName,Long time) {
        this.machineName = machineName;
        this.time = time;
    }

    @Override
    public void run() {
        try {
            getResult();
            Thread.sleep(time);
            System.out.println(Thread.currentThread().getName()+machineName+"Capture done");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public String getResult() {
        return this.machineName+" finish:" + time;
    }
}


```


## 线程打断

```
package threadapi;

public class ThreadInterrupted  {

    private static final Object MONITOR = new Object();

    public static void main(String[] args) {
        try {
           /* Thread thread = new Thread() {
                @Override
                public void run () {
                    while (true) {

                        //Thread.sleep(1000L);
                        System.out.println("1>>" + Thread.currentThread().isInterrupted());
                        synchronized (MONITOR) {
                            try {
                                MONITOR.wait(10);
                            }catch (InterruptedException e) {
                                System.out.println("正在wait的时候被中断一下");
                                e.printStackTrace();
                                return;
                            }
                        }
                    }
                }

            } ;*/

            /*Thread thread = new Thread() {
                @Override
                public void run () {
                    while (true) {

                        try {
                            Thread.sleep(1000L);
                            System.out.println("1>>" + Thread.currentThread().isInterrupted());
                        } catch (InterruptedException e) {
                            System.out.println("sleep状态被打断了");
                            e.printStackTrace();
                            break;
                        }
                    }
                }

            } ;*/

            Thread thread = new Thread() {
                @Override
                public void run () {
                    try {
                        Thread t1 = new Thread(() -> {
                            while (true) {
                                try {
                                    Thread.sleep(1000L);
                                    System.out.println("running");
                                } catch (InterruptedException e) {
                                    e.printStackTrace();
                                }
                            }
                        });

                        t1.setDaemon(true);
                        t1.start();
                        t1.join();
                    } catch (InterruptedException e) {

                        System.out.println("join线程被打断===========");
                        e.printStackTrace();

                    }
                }

            } ;

            thread.start();

            Thread.sleep(1000*10L);
            //如果该线程阻塞的调用wait() ， wait(long) ，或wait(long, int)的方法Object类，
            // 或者在join() ， join(long) ， join(long, int) ， sleep(long) ，或sleep(long, int) ，
            // 这个类的方法，那么它的中断状态将被清除，并且将收到一个InterruptedException 。

            System.out.println(thread.isInterrupted());
            thread.interrupt();
            System.out.println(thread.isInterrupted());


        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
## 关闭线程

### Flag

```
package threadapi;

/**
 * 如何关闭线程
 * 1. 中断
 */
public class ThreadCloseFlag {

    private static class  Worker extends Thread {

        private volatile boolean start = true;

        @Override
        public void run () {
            while (start) {
                //todo
                try {
                    Thread.sleep(1000*1L);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(11);
            }
        }

        public void shutdown() {
            this.start = false;
        }
    }

    public static void main(String[] args) {
        Worker worker = new Worker();
        worker.start();

        try {
            Thread.sleep(1000*10L);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        worker.shutdown();
    }
}
```

### 打断终止

```
package threadapi;

import org.omg.PortableServer.THREAD_POLICY_ID;

public class ThreadCloseInterrupt {

    private static class  Worker extends Thread {


        @Override
        public void run () {
            while (true) {
                //todo
                try {
                    System.out.println(11);
                    Thread.sleep(1000*1L);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                    System.out.println("捕获");
                    //return; //结束
                    break; //有机会做一些事情
                }
                System.out.println(22);
                
                /*if (Thread.interrupted()) {
                    break;
                }*/


            }
            //System.out.println("break");
        }


    }

    public static void main(String[] args) {
       Worker worker = new Worker();
        worker.start();

        try {
            Thread.sleep(1000*10L);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        worker.interrupt();
    }
}
```

## 守护线程结束退出

```
package threadapi;

import java.text.BreakIterator;

public class ThreadService {

    private Thread executeThread;

    private boolean finished = false;

    public void execute(Runnable task) {

        executeThread = new Thread() {
            @Override
            public void run () {
                Thread taskThread = new Thread(task);
                //taskthred是守护线程, 主线程结束之后,守护线程就会结束
                taskThread.setDaemon(true);
                taskThread.start();

                //加入一个join等待任务线程结束,此时主线程也没有结束,守护线程当然也不会退出
                try {
                    taskThread.join();
                    finished = true;
                } catch (InterruptedException e) {
                    System.out.println("任务超时被打断");
                    e.printStackTrace();
                }
                //这种情况就是主线程jion等待守护线程结束,守护线程结束之后,主线程结束;最后都完美结束
            }
        };

        executeThread.start();
    }



    public void shutdown(long mills) {
        long currentTime = System.currentTimeMillis();
        while (!finished) {
            if (System.currentTimeMillis() - currentTime > mills) {
                //任务超时
                System.out.println("任务超时,需要结束");
                executeThread.interrupt();
                break;
            }
            //没有超时也没有执行结束,休眠一下
            try {
                System.out.println("任务还没有超时,再等1s");
                executeThread.sleep(1000L);
            } catch (InterruptedException e) {
                e.printStackTrace();
                break;
            }
        }
        finished = false;
    }

}


package threadapi;

/**
 *  强制退出, 线程blocked,无法获得flag和无法中断
 *  1.中断线程的办法, 封装一个执行线程, 用执行线程建立一个守护线程,守护线程执行任务,如果超时了,直接中断,执行线程,
 *  守护线程就可以结束
 */
public class ThreadCloseForce {

    public static void main(String[] args) {

        ThreadService threadService = new ThreadService();
        long start = System.currentTimeMillis();
        threadService.execute( () -> {
            int i = 0;
            while (true) {
                try {
                    Thread.sleep(1000L);
                    System.out.println(i+":aaaaa");
                    i++;
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        threadService.shutdown(1000*3);
    }
}
```

## 异步数据共享

```
package sync;

public class Bank {
    /**
     * 多线程数据同步问题
     * @param args
     *
     * 1. 利用同步代码块及解决
     */
    public static void main(String[] args) {
        /*final TicketWindowRunnable ticketWindow = new TicketWindowRunnable();
        Thread thread1 = new Thread(ticketWindow, "1号");
        Thread thread2 = new Thread(ticketWindow, "2号");
        Thread thread3 = new Thread(ticketWindow, "3号");
        thread1.start();
        thread2.start();
        thread3.start();*/

        final SynchronizedRunnable ticketWindow = new SynchronizedRunnable();
        Thread thread1 = new Thread(ticketWindow, "1号");
        Thread thread2 = new Thread(ticketWindow, "2号");
        Thread thread3 = new Thread(ticketWindow, "3号");
        thread1.start();
        thread2.start();
        thread3.start();
        /**
         *  为什么只有2号窗口在做事?  同步方法的锁实际上是this,而且加锁的是方法不是代码块
         *
         *  2号拿到锁了,直接执行结束了, 1号和3号执行的时候发现index已经大于max了
         *
         *  实际上是同步执行了代码,
         */


    }
}


package sync;


/**
 * 使用一个runnable实现了,多个线程之间的使用同一份数据 一个实例的管理中心 , 将可执行单元和线程控制分离开来
 * 1. 线程安全代码块,    关键字代码块  synchronized (MONITOR) { }
 */
public class TicketWindowRunnable implements Runnable {

    private int index = 1 ;
    private final int Max = 500;

    private final Object MONITOR = new Object();
    @Override
    public void run() {
        while (true) {
            //代码块执行的时候变成了单线程的,会影响效率,多线程变成了单线程处理
            synchronized (MONITOR) {
                if (index > Max) {
                    break;
                }
                try {
                    Thread.sleep(5);
                    System.out.println(Thread.currentThread().getName() + "current num :" + index++);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

        }
    }
}



//同步方法
package sync;

/**
 * 同步方法,方法用Synchronized 修饰
 * 1. 同步run方法, 实际上只有一个线程在做事
 * 2.
 */
public class SynchronizedRunnable implements Runnable {

    private int index = 1 ;
    private final int Max = 500;

    private final Object MONITOR = new Object();
    @Override
    public void run() {
        while (true) {
           /* //代码块执行的时候变成了单线程的,会影响效率,多线程变成了单线程处理
            if (index > Max) {
                System.out.println(Thread.currentThread().getName());
                break;
            }
            try {
                Thread.sleep(5);
                System.out.println(Thread.currentThread().getName() + "current num :" + index++);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }*/
            if (!hasTicket()) {
                break;
            }

        }
    }

    /**
     * v1 synchronized 加在run方法上,
     * v2 自定义同步方法
     */
    private synchronized boolean hasTicket() {
        //1.getField
        if (index > Max) {
            return false;
        }
        try {
            Thread.sleep(5);
            //1.get Field index
            //2.index = inex +1
            //3.put Field index
            System.out.println(Thread.currentThread().getName() + "current num :" + index++);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return true;

    }

    /**
     * 同步方法是this锁,也可以改写为下面这个
     * @return
     */
/*
    private  boolean hasTicket() {
        synchronized (this) {
            //1.getField
            if (index > Max) {
                return false;
            }
            try {
                Thread.sleep(5);
                //1.get Field index
                //2.index = inex +1
                //3.put Field index
                System.out.println(Thread.currentThread().getName() + "current num :" + index++);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return true;
        }

    }
*/

}


package sync;

/**
 * synchronized测试
 * 1. 三个线程同一把锁
 */
public class SynchronizedTest {

    private final static Object LOCK = new Object();

    public static void main(String[] args) {
           Runnable runnable = () -> {
               synchronized (LOCK) {
                   try {
                       Thread.sleep(10_000);
                       System.out.println(Thread.currentThread().getName()+"test");
                   } catch (InterruptedException e) {
                       e.printStackTrace();
                   }
               }
           };

        Thread t1 = new Thread(runnable);
        Thread t2 = new Thread(runnable);
        Thread t3 = new Thread(runnable);
        long start =System.currentTimeMillis();
        t1.start();
        t2.start();
        t3.start();

        //不加join也是一个一个执行,但是没有顺序,  加了jion顺序执行
        try {
            t1.join();
            t2.join();
            t3.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(System.currentTimeMillis() - start);
        //可以通过jconsole看到,当前monitor的拥有者,只有线程1执行结束,线程2才能开始执行,变成了串行
    }
}


//静态方法锁

package sync;

/**
 * 静态方法锁  synchronized 修饰静态方法代表是 class
 */
public class SynchronizedStatic {

    public synchronized static void m1 () {
        try {
            System.out.println("m1"  + Thread.currentThread().getName());
            Thread.sleep(5_000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void m2 () {
        synchronized(SynchronizedStatic.class) {
            try {
                System.out.println("m2"  + Thread.currentThread().getName());
                Thread.sleep(5_000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

package sync;

public class SynchronizedStaicTest {

    public static void main(String[] args) {

        new Thread("t1") {
            @Override
            public void run () {
                SynchronizedStatic.m1();
            }

        }.start();

        new Thread("t2") {
            @Override
            public void run () {
                SynchronizedStatic.m2();
            }

        }.start();
    }
}

```

# 死锁


```
package DeadLock;

public class DeadLock {


    private OtherService  otherService;

    private final Object LOCK = new Object();

    public DeadLock(OtherService otherService) {
        this.otherService = otherService;
    }

    public void m1 () {
        synchronized (LOCK) {
            System.out.println("m1" + Thread.currentThread().getName());
            otherService.s1();
        }

    }

    public void m2 () {
        synchronized (LOCK) {
            System.out.println("m2" + Thread.currentThread().getName());
        }
    }
}


package DeadLock;

public class OtherService {

    private final Object LOCK2 = new Object();

    private DeadLock deadLock;

    public void setDeadLock(DeadLock deadLock) {
        this.deadLock = deadLock;
    }

    public void s1() {
        synchronized (LOCK2) {
            System.out.println("s1" + Thread.currentThread().getName());
        }
    }

    public void s2() {
        synchronized (LOCK2) {
           deadLock.m2();
        }
    }
}


package DeadLock;

public class DeadLockTest {

    public static void main(String[] args) {
        OtherService otherService = new OtherService();
        DeadLock deadLock = new DeadLock(otherService);
        otherService.setDeadLock(deadLock);

        new Thread("T1") {
            @Override
            public void run () {
                while (true) {
                    deadLock.m1();
                }
            }
        }.start();

        new Thread("T2") {
            @Override
            public  void  run () {
                while (true) {
                    otherService.s2();
                }
            }
        }.start();

    }
}

通过jps  就stack查看进程是不是有死锁

```

## 线程异常

```
package ThreadException;

/**
 *  线程里面是无法抛出异常的, 线程在main里面启动, 线程是无法抛出异常
 *
 *   在线程外知道线程异常
 */

public class ThreadException {

    private final static int A = 10;
    private final static int B = 0;

    public static void main(String[] args) {
        Thread thread = new Thread(() -> {
            try {
                Thread.sleep(5_000);
                //这里是重写run方法, run方法是无法抛出异常的
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
    }
}

```



