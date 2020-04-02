# ThreadLocal

```
package ThreadLocal;

/**
 *  这个类提供线程局部变量。 这些变量与其正常的对应方式不同，
 *  因为访问一个的每个线程（通过其get或set方法）都有自己独立初始化的变量副本。
 *
 *  ThreadLocal<T>  泛型方法设置,如果设置了T 获取T不需要强制类型转换
 */
public class ThreadLocalSimpleTest {

    private static ThreadLocal<String> threadLocal = new ThreadLocal() {

        //设置value的默认值
        @Override
        protected String initialValue () {
            return "Alex";
        }

    };

    public static void main(String[] args) {
        threadLocal.set("name");
        threadLocal.get();
        System.out.println(threadLocal.get());
    }
}

package ThreadLocal;

import java.util.Random;

/**
 * 这个类提供线程局部变量。 这些变量与其正常的对应方式不同，
 *  因为访问一个的每个线程（通过其get或set方法）都有自己独立初始化的变量副本。
 *
 *  多个线程操作一份同一个threadlocal实际上这个threadlocal是属于自己的线程的局部变量,
 *  不能被其他线程访问,也不能刚被其他线程修改
 */
public class ThreadLocalComplexTest {

    private static final ThreadLocal<String> threadLocal = new ThreadLocal<>();

    public static void main(String[] args) {

        Random random = new Random(System.currentTimeMillis());

        Thread thread = new Thread( () -> {
            try {
                threadLocal.set("Thread-01");
                Thread.sleep(random.nextInt(1000));
                System.out.println(Thread.currentThread().getName() + ":"+ threadLocal.get());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        thread.start();

        Thread thread2 = new Thread( () -> {
            try {
                threadLocal.set("Thread-02");
                Thread.sleep(random.nextInt(1000));
                System.out.println(Thread.currentThread().getName() + ":" + threadLocal.get());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        thread2.start();

        try {
            thread.join();
            thread2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }


        threadLocal.set("Main");
        System.out.println(Thread.currentThread().getName() + ":" + threadLocal.get());
    }
}

package ThreadLocal;


import com.sun.org.apache.regexp.internal.RE;
import jdk.nashorn.internal.ir.ReturnNode;
import sun.awt.SunHints;

import java.util.HashMap;
import java.util.Map;

/**
 * 模式Threadlocal
 */
public class ThreadLocalSimulator<T> {

    private final Map<Thread,T> storage = new HashMap<>();

    public void  set(T t) {
        synchronized (this) {
            Thread key = Thread.currentThread();
            storage.put(key,t);
        }
    }

    public T  get() {
        synchronized (this) {
            Thread key = Thread.currentThread();
            T value = storage.get(key);
            if (value == null)
                return initValue();
            return value;
        }
    }

    private T initValue() {
        return null;
    }
}


package ThreadLocal;

import java.util.Random;

public class ThreadLocalSimulaorTest {

    private static final ThreadLocalSimulator<String> threadLocal = new ThreadLocalSimulator<String>();

    public static void main(String[] args) {

        Random random = new Random(System.currentTimeMillis());

        Thread thread = new Thread( () -> {
            try {
                threadLocal.set("Thread-01");
                Thread.sleep(random.nextInt(1000));
                System.out.println(Thread.currentThread().getName() + ":"+ threadLocal.get());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        thread.start();

        Thread thread2 = new Thread( () -> {
            try {
                threadLocal.set("Thread-02");
                Thread.sleep(random.nextInt(1000));
                System.out.println(Thread.currentThread().getName() + ":" + threadLocal.get());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        thread2.start();

        try {
            thread.join();
            thread2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }


        System.out.println("======Main=====");
        System.out.println(Thread.currentThread().getName() + ":" + threadLocal.get());
    }
}

```


# ThreadLocal实现Context上下文

```

```
