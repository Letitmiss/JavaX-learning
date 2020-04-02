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
package ThreadLocalCase;

public class Context {

    private String name;
    private String card;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setCard(String card) {
        this.card = card;
    }

    public String getCard() {
        return card;
    }
}

package ThreadLocalCase;

/**
 * Context上下文对象,下一部分,需要上一部分的部分数据,需要传递上下对象
 */
public class ExecutionTask implements Runnable {

    private QueryFromDBAction queryAction =  new QueryFromDBAction();

    private QueryFromHttpAction queryFromHttpAction = new QueryFromHttpAction();

    @Override
    public void run() {
        Context context = new Context();
        queryAction.execute(context);
        queryFromHttpAction.execute(context);
        System.out.println("The name: "+context.getName()+",the card: "+ context.getCard());
    }
}


package ThreadLocalCase;

public class QueryFromDBAction {

    public void  execute(Context context) {

        try {
            Thread.sleep(1000L);
            String name = Thread.currentThread().getName()+"-tom";
            context.setName(name);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

    }

    public void  execute() {

        try {
            Thread.sleep(1000L);
            String name = Thread.currentThread().getName()+"-tom";
            ActionContext.getActionContext().getContext().setName(name);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

    }
}


package ThreadLocalCase;

public class QueryFromHttpAction {

    public void execute (Context context) {

        try {
            Thread.sleep(1000L);
            String name = context.getName();
            String card = getCard(name);
            context.setCard(card);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

    }

    public void execute () {

        try {
            Thread.sleep(1000L);
            String name = ActionContext.getActionContext().getContext().getName();
            String card = getCard(name);
            ActionContext.getActionContext().getContext().setCard(card);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

    }

    private String getCard(String name) {

        try {
            Thread.sleep(2_000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "12345667-"+ Thread.currentThread().getName();
    }
}

```

## 改造Context

```
package ThreadLocalCase;

public class ActionContext {

    private static final ThreadLocal<Context> THREAD_LOCAL = new ThreadLocal<Context>() {
        @Override
        protected Context initialValue() {
            return new Context();
        }
    };

    private static class InstanceHolder {
       private final static ActionContext ACTION_CONTEXT = new ActionContext();
    }

    public static ActionContext getActionContext() {
        return InstanceHolder.ACTION_CONTEXT;
    }

    public Context getContext() {
        return THREAD_LOCAL.get();
    }
}

package ThreadLocalCase;

/**
 * Context上下文对象,下一部分,需要上一部分的部分数据,需要传递上下对象
 */
public class ExecutionTask implements Runnable {

    private QueryFromDBAction queryAction =  new QueryFromDBAction();

    private QueryFromHttpAction queryFromHttpAction = new QueryFromHttpAction();

   /* @Override
    public void run() {
        Context context = new Context();
        queryAction.execute(context);
        queryFromHttpAction.execute(context);
        System.out.println("The name: "+context.getName()+",the card: "+ context.getCard());
    }*/

    @Override
    public void run() {
        queryAction.execute();
        queryFromHttpAction.execute();
        System.out.println("The name: " + ActionContext.getActionContext().getContext().getName() +
                ",the card: " + ActionContext.getActionContext().getContext().getCard());
    }
}


package ThreadLocalCase;

import java.util.stream.IntStream;

public class ContextTest {

    public static void main(String[] args) {

        IntStream.rangeClosed(1,5).forEach( i -> {
            new Thread(new ExecutionTask()).start();
        });
    }
}


```
