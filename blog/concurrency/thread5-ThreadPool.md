# 线程池
```
package ThreadPool;

import java.util.ArrayList;
import java.util.LinkedList;
import java.util.List;

public class SimpleThreadPool {

    private final int size;
    private final static int DEFAULT_SIZE = 10;

    private static int seq = 0 ;
    private final static String THREAD_PREFIX = "SIMPLE_THREAD_POOL-";
    private final static ThreadGroup GROUP = new ThreadGroup("THREAD_POOL");

    private final static LinkedList<Runnable> TASK_QUEUE = new LinkedList<>();

    private final static List<WorkerTask> THREAD_QUEUE = new ArrayList<>();

    private static int taskNum = 0;

    public SimpleThreadPool() {
        this.size = DEFAULT_SIZE;
        init();
    }

    private void init() {
        for (int i = 0; i < size; i++) {
            createWorkTask();
        }
    }

    private enum TaskState {
        FREE,RUNNING,BLOCKED,DEAD
    }


    public void submit(Runnable runnable) {
        synchronized (TASK_QUEUE) {
            TASK_QUEUE.addLast(runnable);
            TASK_QUEUE.notifyAll();
        }
    }

    private void createWorkTask() {
        WorkerTask workerTask = new WorkerTask(GROUP, THREAD_PREFIX + seq++);
        workerTask.start();
        THREAD_QUEUE.add(workerTask);
    }


    private static class WorkerTask extends Thread {

        private static volatile TaskState  taskState= TaskState.FREE;

        public WorkerTask(Runnable target, String name) {
            super(target, name);
        }

        public WorkerTask(ThreadGroup group, String name) {
            super(group,name);
        }

        public TaskState getTaskState () {
            return taskState;
        }

        public void close () {
            taskState = TaskState.DEAD;
        }

        public void run() {
            //label
            OUTER:
            while (taskState != TaskState.DEAD) {
                Runnable runnable = null;
                synchronized (TASK_QUEUE) {
                    //队列为空
                    while (TASK_QUEUE.isEmpty()) {
                        try {
                            taskState = TaskState.BLOCKED;
                            TASK_QUEUE.wait();
                            System.out.println(Thread.currentThread().getName() + " wake up");
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                            break OUTER;
                        }
                    }
                    //队列不为空
                    runnable = TASK_QUEUE.removeFirst();

                }
                if (runnable != null) {
                    System.out.println("taskNum : " + ++taskNum);
                    taskState = TaskState.RUNNING;
                    runnable.run();
                    taskState = TaskState.FREE;
                }
            }
        }
    }
}

```
