# 线程通信
```
package Communication;

import java.util.PrimitiveIterator;

public class ProduceConsumerVersion1 {

    private int i= 1;
    final Object LOCK = new Object();

    private void  produce () {
        synchronized (LOCK) {
            System.out.println("P->"+i++);
        }
    }

    private void consumer() {
        synchronized (LOCK) {
            System.out.println("C->" + i);
        }
    }


    public static void main(String[] args) {

        ProduceConsumerVersion1 pc = new ProduceConsumerVersion1();
        /**
         * 问题:
         * 1. 线程1一直在生产,不管线程2有没有消费,
         * 2. 线程2一直消费, 消费的是最新, 有很多没有消费
         *
         * 希望现象:
         * 线程1生产1一个,线程2消费一个, 如果线程2没有消费, 线程1就不生产, 需要线程1和线程2通信,
         *
         */
        new Thread( () -> {
            while (true) {
                pc.produce();
            }
        },"produce").start();

        new Thread( () -> {
            while (true) {
                pc.consumer();
            }
        },"consumer").start();
    }
}


package Communication;

/**
 * p 生产 - c消费
 *
 * p生产1个,如果没有消费,就不生产
 * C消费一个,如果没有生产的自己等待
 *
 * wait  notify 等待和通知线程之间的通信
 *
 * 1. 一个生产一个消费的时候这个模型是OK的
 * 2. 如果有多个生产和多个消费者, 这个模型就会有问题
 */
public class ProduceConsumerVersion2 {

    private int i= 0;
    final Object LOCK = new Object();

    //加入控制变量
    private volatile boolean isProduct = false;

    private void  produce () {
        synchronized (LOCK) {
            if (isProduct) {
                try {
                    LOCK.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            } else {
                i++;
                System.out.println("P->" + i);
                isProduct = true;
                LOCK.notify();
            }
        }
    }

    private void consumer() {
        synchronized (LOCK) {
            if (isProduct) {
                System.out.println("C->" + i);
                isProduct = false;
                LOCK.notify();
            } else {
                try {
                    LOCK.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }


    public static void main(String[] args) {

        ProduceConsumerVersion2 pc = new ProduceConsumerVersion2();

        new Thread( () -> {
            while (true) {
                pc.produce();
            }
        },"produce").start();

        new Thread( () -> {
            while (true) {
                pc.consumer();
            }
        },"consumer").start();
    }
}


package Communication;

import java.util.stream.Stream;

/**
 * p 生产 - c消费
 *
 * p生产1个,如果没有消费,就不生产
 * C消费一个,如果没有生产的自己等待
 *
 * wait  notify 等待和通知线程之间的通信
 *
 * 1. 一个生产一个消费的时候这个模型是OK的
 * 2. 如果有多个生产和多个消费者, 这个模型就会有问题
 *
 * 问题就是执行过程中, 线程都会进入waiting状态, 因为notify通知的其实就是lOCK锁的其他人,
 * 不管是谁,只要是有这个锁其他线程都会通知
 */
public class ProduceConsumerVersion3 {

    private int i= 0;
    final Object LOCK = new Object();

    //加入控制变量
    private volatile boolean isProduct = false;

    private void  produce () {
        synchronized (LOCK) {
            while (isProduct) {
                try {
                    LOCK.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            i++;
            System.out.println(Thread.currentThread().getName()+" : P->" + i);
            isProduct = true;
            LOCK.notifyAll();

        }
    }

    private void consumer() {
        synchronized (LOCK) {
            while (!isProduct) {
                try {
                    LOCK.wait();
                    //如果被环境,
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println(Thread.currentThread().getName()+" : C->" + i);
            isProduct = false;
            LOCK.notifyAll();

        }
    }

    public static void main(String[] args) {

        ProduceConsumerVersion3 pc = new ProduceConsumerVersion3();
        Stream.of("P1","P2").forEach( p -> {
            new Thread( () -> {
                while (true) {
                    pc.produce();
                }
            },p).start();
        });

        Stream.of("C1","C2").forEach( c -> {
            new Thread( () -> {
                while (true) {
                    pc.consumer();
                }
            },c).start();
        });

    }
}



package Communication;

import threadapi.ThreadAPI;

import java.util.stream.Stream;

/**
 *  sleep 与 wait的不同
 *
 *  1. sleep是Thread是方法 ,  wait是Objec方法
 *  2. sleep 没有释放锁, 但是 wait释放了锁, 并且把线程加入了对象的monitor队列中
 *  3. sleep 方法不需要依赖monitor, 但是wait需要依赖, 必须要synchronized关键字
 *  4. sleep 不需要被唤醒, 但是wait是需要被唤醒的
 */
public class ThreadWaitSleep {

    private final static Object MONITOR = new Object();

    public static void main(String[] args) {
        //m1();
        //m2();

        Stream.of("T1","T2").forEach( p -> {
            new Thread( () -> {
                //m3(); 有明显的先后顺序
                m2(); //基本同时输出,释放了锁
            },p).start();
        });
    }


    public static void m1() {
        try {
            System.out.println("The thread " + Thread.currentThread().getName());
            Thread.sleep(10_000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    /**
     * 使用monitor加锁, 因为wait的是monitor
     */

    public static void m2() {
        synchronized (MONITOR) {
            try {
                System.out.println("The thread " + Thread.currentThread().getName());
                MONITOR.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void m3() {
        synchronized (MONITOR) {
            try {
                System.out.println("The thread " + Thread.currentThread().getName());
                Thread.sleep(10_000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

}



package Communication;

import java.net.ConnectException;
import java.util.*;

/**
 * 多线程,全部采集,所有子线程结束,总任务才算结束
 *
 * 1. 不一定线程越多,就性能越高, 线程太多,消费CPU进行上下文的切换
 */
public class CommunicationCase {

    private static LinkedList<control>   CONTROLS= new LinkedList<>();

    private static int MAX = 5;

    public static void main(String[] args) {

        List<Thread>  worker = new ArrayList<>();
        Arrays.asList("M1","M2","M3","M4","M5","M6","M7","M8","M9","M10").stream()
                .map(CommunicationCase::createCaptureThread)
                .forEach( t -> {
                    t.start();
                    worker.add(t);
                    }
                );

        worker.stream().forEach(thread -> {
            try {
                thread.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });


        Optional.of("All of capture work finished").ifPresent(System.out:: println);
    }


    private static Thread createCaptureThread(String name) {
        return new Thread( () -> {
            Optional.of("The worker ["+ Thread.currentThread().getName() + "]  to enter")
                    .ifPresent(System.out::println);
            synchronized (CONTROLS) {
                while (CONTROLS.size() > MAX) {
                    try {
                        CONTROLS.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                CONTROLS.addLast(new control());
            }

            Optional.of("The worker [" + Thread.currentThread().getName() + "] to worker")
                    .ifPresent(System.out::println);

            try {
                Thread.sleep(10_000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            synchronized (CONTROLS) {
                Optional.of("The worker [" + Thread.currentThread().getName() + "] to stop")
                        .ifPresent(System.out::println);
                CONTROLS.removeFirst();
                CONTROLS.notifyAll();
            }

        },name);
    }

    private static class control {

    }
}


```
