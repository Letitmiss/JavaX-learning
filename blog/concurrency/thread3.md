# 自定义锁

```
package CustomLock;

import java.util.Collection;

public interface Lock {

    static class TimeOutExcetpion extends Exception {

        public TimeOutExcetpion(String message) {
            super(message);
        }
    }

    void lock() throws InterruptedException;

    void lock(long mills) throws  InterruptedException,TimeOutExcetpion;

    void unlock();

    Collection<Thread> getBlockedThread();

    int getBlockSize();
}


package CustomLock;

import java.util.ArrayList;
import java.util.Collection;
import java.util.Collections;

public class BooleanLock implements Lock {

    //true , 锁被拿走了
    //false, 锁是闲着的
    private boolean initValue;

    private Thread currentThread;

    private Collection<Thread> blockedThreadCollection = new ArrayList<>();

    public BooleanLock() {
        this.initValue = false;
    }

    @Override
    public synchronized void lock() throws InterruptedException {
        while (initValue) {
            blockedThreadCollection.add(Thread.currentThread());
            this.wait();
        }
        blockedThreadCollection.remove(Thread.currentThread());
        this.initValue = true;

        this.currentThread = Thread.currentThread();
    }

    @Override
    public synchronized void lock(long mills) throws InterruptedException, TimeOutExcetpion {
        if (mills <= 0)
            lock();

        long hasRemaining = mills;
        long endTime = System.currentTimeMillis()+ mills;

        while (initValue) {
            if (hasRemaining <= 0)
                throw new TimeOutExcetpion("time out");
            blockedThreadCollection.add(Thread.currentThread());
            this.wait(mills);

            hasRemaining = endTime - System.currentTimeMillis();
        }

        this.initValue = true;
        this.currentThread = Thread.currentThread();

    }

    @Override
    public synchronized void unlock() {
        if (Thread.currentThread() == this.currentThread) {
            this.initValue = false;
            System.out.println(Thread.currentThread().getName() + " release the lock");
            this.notifyAll();
        }
    }

    @Override
    public Collection<Thread> getBlockedThread() {
        return Collections.unmodifiableCollection(blockedThreadCollection);
    }

    @Override
    public int getBlockSize() {
        return 0;
    }
}

package CustomLock;


import java.util.Optional;
import java.util.stream.Stream;

/**
 * http://ifeve.com/doug-lea/
 * http://ifeve.com/category/java/
 */
public class LockTest {

    public static void main(String[] args) {
        final BooleanLock  booleanLock = new BooleanLock();

        Stream.of("T1","T2","T3","T4").forEach( p -> {
            new Thread( () -> {
                try {
                    //booleanLock.lock();
                    booleanLock.lock(1000);
                    System.out.println(Thread.currentThread().getName() + " get lock ");
                    worker();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (Lock.TimeOutExcetpion e) {
                    e.printStackTrace();
                    System.out.println(Thread.currentThread().getName()  + " time out");
                } finally {
                    booleanLock.unlock();
                }
            },p).start();
        });

        try {
            Thread.sleep(1_000);
            booleanLock.unlock();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    private static void worker () {
        Optional.of(Thread.currentThread().getName() + " is working").ifPresent(System.out::println);
        try {
            Thread.sleep(10_000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}




package CustomLock;

public class LockTest2 {

    public static void main(String[] args) throws InterruptedException {

        Thread t1 = new Thread(() -> {
            run();
        }, "t1");
        t1.start();

        Thread.sleep(2_000);

        Thread t2 = new Thread(() -> {
            //如果超时就不要执行退出,没必要等待
            //强锁,如果超时就退出
            run();
            
        }, "t2");
        t2.start();

        Thread.sleep(2_000);
        t1.interrupt();



        /**
         * 如果某个线程抢到锁,一直不释放,其他线程无法获得锁
         * 1.可以通过打断t1,让t2获得锁,
         *
         * 2. 不打断t1,不影响t1的执行, 打断t2,打断但是t2没有停止,
         *      设置如果t2等一定时间,如果拿不到锁就做其他事,不要如此执着
         */

    }


    private synchronized static void run (){
        System.out.println(Thread.currentThread().getName()+ " is running ");
        while (true) {
           /* try {
                Thread.sleep(1_000);
            } catch (InterruptedException e) {
                System.out.println("被打断了");
                e.printStackTrace();
                break;
            }*/
        }
    }
}




```
