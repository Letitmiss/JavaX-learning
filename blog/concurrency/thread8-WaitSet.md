# Wait Set

```
package WaitSet;

import java.util.Optional;
import java.util.stream.IntStream;

/**
 * Wait放弃CPU的执行权,
 *    放弃CPU的执行权, 存在哪里, 如何被唤醒的
 *
 * 1. 所有对象都有一个 wait set ,用来存放调用了对象的wait方法之后进入block状态线程;
 * 2. 线程被notify之后,不一定立即执行;
 * 3. 线程从wait中被唤醒顺序不一定是FIFO 先进先出;
 */
public class WaitSet {

    private static final Object LOCK = new Object();

    private static void  worker () {
        synchronized (LOCK)  {
            System.out.println("Begin....");

            try {
                System.out.println("Thread will comming");
                //wait时候是释放锁了, 被唤醒,也要竞争锁,但是执行不会从头开始执行,
                LOCK.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("Thread will out.");
        }
    }

    public static void main(String[] args) throws InterruptedException {

       /* IntStream.rangeClosed(1,10).forEach( i -> {
            new Thread(String.valueOf(i)) {
                @Override
                public void run () {
                    synchronized (LOCK) {
                        try {
                            Optional.of(Thread.currentThread().getName() + " will wait set ").ifPresent(System.out::println);
                            LOCK.wait();
                            Optional.of(Thread.currentThread().getName() + " leave  wait set ").ifPresent(System.out::println);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }
            }.start();
        });

        Thread.sleep(1_000);

        IntStream.rangeClosed(1,10).forEach(i-> {
            synchronized (LOCK) {
                LOCK.notify();
                try {
                    Thread.sleep(1_000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });*/


        //启动一个线程
        new Thread( () -> {
            worker();
        }).start();

        Thread.sleep(1_000);

        /*new Thread( () -> {
            synchronized (LOCK) {
                LOCK.notify();
            }
        }).start();*/

        synchronized (LOCK) {
            LOCK.notify();
        }


    }
}

```
