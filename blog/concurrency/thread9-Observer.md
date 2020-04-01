# 观察者模式

```
package Observertest;

import java.util.ArrayList;
import java.util.List;

/**
 * 主题: 如果数据有变化,通知所有Observer
 */
public class Subject {

    private List<Observer> observerList = new ArrayList<>();

    private int state;

    public int getState() {
        return this.state;
    }

    public void setState (int state) {
        if (state == this.state) {
            return;
        }
        this.state = state;
        notifyObserver();
    }

    public void attach(Observer observer) {
        observerList.add(observer);
    }


    private void notifyObserver() {
        observerList.forEach(Observer::update);
    }
}

package Observertest;

/**
 * 抽象的Observer
 */
public abstract  class Observer {

    protected Subject subject;

    public Observer(Subject subject) {
        this.subject = subject;
        this.subject.attach(this);
    }

     abstract void update();
}

package Observertest;

public class BinaryObserver extends Observer {

    public BinaryObserver(Subject subject) {
        super(subject);
    }

    @Override
    void update() {
        System.out.println("Binary String : " + Integer.toBinaryString(subject.getState()));

    }
}


package Observertest;

public class OctalObserver extends Observer {

    public OctalObserver(Subject subject) {
        super(subject);
    }

    @Override
    void update() {
        System.out.println("Octal string : " + Integer.toOctalString(subject.getState()));
    }
}


package Observertest;

public class ObseverClient {

    public static void main(String[] args) {

        final Subject subject = new Subject();

       BinaryObserver binaryObserver = new BinaryObserver(subject);
       OctalObserver octalObserver = new OctalObserver(subject);

       System.out.println("---------------");
       subject.setState(10);

       System.out.println("---------------");
       subject.setState(16);
        
    }
}

```
## 线程观察者模式

```
package Observertest;

public abstract class ObservableRunnable implements Runnable {

    final protected LifeCycleListener listener;

    public ObservableRunnable(final LifeCycleListener listener) {
        this.listener = listener;
    }

    protected void notifyChange (final RunnableEvent runnableEvent) {
        listener.onevent(runnableEvent);
    }

    public enum RunnableState {
        RUNNING,ERROR,DONE;
    }

    public static class RunnableEvent {
        private final RunnableState state;
        private final Thread thread;
        private final Throwable cause;

        public RunnableEvent(RunnableState state, Thread thread, Throwable cause) {
            this.state = state;
            this.thread = thread;
            this.cause = cause;
        }

        public RunnableState getState() {
            return state;
        }

        public Thread getThread() {
            return thread;
        }

        public Throwable getCause() {
            return cause;
        }
    }
}


package Observertest;

public interface LifeCycleListener {

    void onevent(ObservableRunnable.RunnableEvent runnableEvent);
}


package Observertest;

import java.util.List;

public class ThreadLifeCycleListener implements LifeCycleListener {

    private final Object LOCK = new Object();

    public void concurrency(List<String> ids) {
        if (ids == null || ids.isEmpty())
            return;
        ids.stream().forEach( id -> {
            new Thread(new ObservableRunnable(this) {
                @Override
                public void run() {
                    try {
                        notifyChange(new RunnableEvent(RunnableState.RUNNING,Thread.currentThread(),null));
                        System.out.println("query for the id " + id);
                        if ("2".equals(id)) {
                            int a= 10 /0;
                        }
                        Thread.sleep(1000L);
                        notifyChange(new RunnableEvent(RunnableState.DONE,Thread.currentThread(),null));
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                        notifyChange(new RunnableEvent(RunnableState.ERROR,Thread.currentThread(),e));
                    } catch (Exception e) {
                        e.printStackTrace();
                        notifyChange(new RunnableEvent(RunnableState.ERROR,Thread.currentThread(),e));
                    }
                }
            }).start();
        });
    }

    @Override
    public void onevent(ObservableRunnable.RunnableEvent runnableEvent) {
        synchronized (LOCK) {
            System.out.println("The Runnable [" + runnableEvent.getThread().getName()+"] data changed and state is ["+runnableEvent.getState()+"]");
            if (runnableEvent.getCause() != null) {
                System.out.println("The runnable [" + runnableEvent.getThread().getName() + "] process failed");
                runnableEvent.getCause().printStackTrace();
            }

        }
    }
}

package Observertest;

import java.util.Arrays;

public class ThreadLifeCycleClient {
    public static void main(String[] args) {

        final ThreadLifeCycleListener threadLifeCycleListener = new ThreadLifeCycleListener();

        threadLifeCycleListener.concurrency(Arrays.asList("1","2"));
    }
}

```
