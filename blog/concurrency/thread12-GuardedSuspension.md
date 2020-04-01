# Guarded Suspension

Guarded Suspension 线程保护挂起模式,等待模式

```
package GuardSuspension;

import java.util.LinkedList;

public class RequestQueue {

    final private LinkedList<Request> requests = new LinkedList<>();


    public LinkedList<Request> getRequests() {
        return requests;
    }

    public Request getRequest() {
        synchronized (requests) {
            while (requests.size() == 0) {
                try {
                    requests.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                    return null;
                }
            }
            return requests.removeFirst();
        }
    }

    public void putRequests(Request request) {
        synchronized (requests) {
           requests.addLast(request);
           requests.notifyAll();
        }
    }
}

package GuardSuspension;

public class Request {

    final private String value;

    public Request(String value) {
        this.value = value;
    }

    public String getValue() {
        return value;
    }
}




package GuardSuspension;

import java.util.Random;

public class ClientThread extends Thread {

    final private RequestQueue queue;

    final private Random random;

    final private String value;

    public ClientThread(RequestQueue queue,String value) {
        this.queue = queue;
        this.value = value;
        this.random = new Random(System.currentTimeMillis());
    }

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println("Client -> request " + value + i);
            queue.putRequests(new Request(value+i));
            try {
                Thread.sleep(random.nextInt(100));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}


package GuardSuspension;

import java.util.Random;

public class ServerThread extends Thread {

    private final RequestQueue queue;

    private final Random random;

    private volatile boolean flag = true;

    public ServerThread(RequestQueue queue) {
        this.queue = queue;
        this.random = new Random(System.currentTimeMillis());
    }

    @Override
    public void run () {
        while (flag) {
            Request request = queue.getRequest();
            if ( null == request) {
                System.out.println("Receive the empty request");
                break;
            }
            System.out.println("Server -> request " + request.getValue());
            try {
                Thread.sleep(random.nextInt(1000));
            } catch (InterruptedException e) {
                return;
            }
        }
    }

    public void close () {
        this.flag = false;
        //有可能线程已经被wait住了,如果不加入唤醒无法判断flag的
        this.interrupt();
    }
}


package GuardSuspension;

/**
 * Suspension 挂起
 */
public class SuspensionClient {

    public static void main(String[] args) throws InterruptedException {
        final RequestQueue requestQueue = new RequestQueue();

        new ClientThread(requestQueue, "Hello").start();
        ServerThread serverThread = new ServerThread(requestQueue);
        serverThread.start();


        Thread.sleep(15_000);
        //没有任务主动关闭server
        serverThread.close();
    }
}

```
