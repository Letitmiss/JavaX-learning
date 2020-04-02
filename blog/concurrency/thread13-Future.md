# Future

```
package Future;

public interface Future<T> {

    T get();
}


package Future;

public interface FutureTask<T> {

    T call();
}


package Future;

import java.util.function.Consumer;

public class FutureService {

    public <T> Future<T>  submit(FutureTask<T> task) {
        AsyncFuture<T> asyncFuture = new AsyncFuture<T>();
        new Thread(() -> {
            T result = task.call();
            asyncFuture.done(result);
        }).start();
        return asyncFuture;
    }

    //增加消费者
    public <T> Future<T>  submit(FutureTask<T> task, Consumer<T> consumer) {
        AsyncFuture<T> asyncFuture = new AsyncFuture<T>();
        new Thread(() -> {
            T result = task.call();
            asyncFuture.done(result);
            consumer.accept(result);
        }).start();
        return asyncFuture;
    }
}


package Future;

public class AsyncFuture<T> implements Future<T> {

    private volatile boolean done = false;
    private T t;

    public void done(T t) {
        synchronized (this) {
            this.t = t;
            this.done = true;
            this.notifyAll();
        }
    }

    @Override
    public T get() {
        synchronized (this) {
            try {
                while (!done) {
                    this.wait();
                }
            }catch (InterruptedException e) {
                e.printStackTrace();
            }
            return t;
        }
    }
}


package Future;

import jdk.nashorn.internal.ir.IfNode;

/**
 * future 设计模式实现 :
 *   异步的通信,调用方法不发生阻塞,自己可以做某些事,过会儿拿结果
 *
 *
 * Future      代表的是未来的一个凭据
 * FutureTask  将调用逻辑进行了分离
 * FutureService 桥接  Future 和 FutureTask
 */
public class SyncInvoker {

    public static void main(String[] args) throws InterruptedException {
        //getData方法非常耗时

        /*long start = System.currentTimeMillis();
        String result = getData();
        System.out.println( " Get "+ result + " time : " + (System.currentTimeMillis() -start));*/

        /*FutureService futureService = new FutureService();
        Future<String> future = futureService.submit(() -> {
            try {
                Thread.sleep(5_000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "FINISH";
        });*/



        FutureService futureService = new FutureService();
        Future<String> future = futureService.submit(() -> {
            try {
                Thread.sleep(5_000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "FINISH";
        }, System.out::println);


        //do other thing
        System.out.println("======strat=======");
        Thread.sleep(1_000);
        System.out.println("=====end=====");

        //get result
        System.out.println("=====get======");
        //System.out.println(future.get());  //如果还没执行结束就会被wait,提早请求

        //优化方式,增加Callback,使用java8的 Consumer

    }

    private static String getData() {
        try {
            Thread.sleep(10_000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "FINISH";
    }
}




```
