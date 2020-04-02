# 读写分离锁 

```
package ReadWriteLock;

/**
 * 读写锁分离
 * 1. 很多线程写操作,加写操作锁, 保证每次写的数据需要拿锁操作
 * 2. 很多线程读取数据,如果加入了锁,获取操作需要等待,串行化,执行效率降低,大量读操作比较多时效率很低;
 *
 * 解决办法:
 *  读写使用不同锁,读写分离,
 *  引入问题:读数据过程中,不能修改, 冲突处理
 *  1. read read   并行化
 *  2. read write  不允许,读的过程中不可以写
 *  3. write write 不允许,写操作不可以并行
 *  4. write read  不允许,
 *
 *
 *  +----------------------------+
 *          READ  | WRITE
 *  ------------------------------
 *  READ  |  Y    |   N
 *  WRITE |  N    |   N
 *  ------------------------------
 *  结论 : 多个线程读操作可以串行,实现读写分离
 */
public class ReadWriteLock {


    private int readingReader = 0;
    private int waitingReader = 0;
    private int writingWriter = 0;
    private int waitingWriter = 0;

    private boolean preferWriter = true;

    public ReadWriteLock(boolean preferWriter) {
        this.preferWriter = preferWriter;
    }

    public ReadWriteLock() {
        this(true);
    }

    public synchronized void readLock () {
        this.waitingReader++;
        try {
            while (waitingWriter > 0 || (preferWriter && waitingWriter > 0)) {
                this.wait();
            }
            this.readingReader++;
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            this.waitingReader--;
        }
    }

    public synchronized void readUnlock() {
        this.readingReader--;
        this.notifyAll();
    }


    public synchronized void writeLock (){
        this.waitingWriter++;
        try {
            while (readingReader > 0 || writingWriter > 0) {
                //正在读的全部wait了,再重新写
                this.wait();
            }
            this.writingWriter++;
        } catch (InterruptedException e) {
             e.printStackTrace();
        } finally {
            this.waitingWriter--;
        }

    }

    public synchronized void writeUnLock () {
        this.writingWriter--;
        this.notifyAll();
    }

}


package ReadWriteLock;

public class ShareData {

    private final char[] buffer;

    private final ReadWriteLock lock = new ReadWriteLock();

    public ShareData(int size) {
        this.buffer = new  char[size];
        for (int i = 0; i < size ; i++) {
           this.buffer[i] = '*';
        }
    }

    public char[] read() {
        try {
            lock.readLock();
            System.out.println(Thread.currentThread().getName() + " read  " + String.valueOf(buffer));
            return doRead();
        } finally {
            lock.readUnlock();
        }
    }

    private char[] doRead() {
        char[] newBuffer = new char[buffer.length];
        for (int i = 0; i < buffer.length; i++) {
            newBuffer[i] = buffer[i];
        }

        slowly(50);
        return newBuffer;
    }

    private void slowly(int i) {
        try {
            Thread.sleep(i);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public void write (char c) {
        try {
            lock.writeLock();
            System.out.println("write char is " + String.valueOf(c));
            doWrite(c);
        } finally {
            lock.writeUnLock();
        }
    }
    private void doWrite(char c) {
        for (int i = 0; i < buffer.length ; i++) {
             buffer[i] = c;
             slowly(i);
        }
    }
}

package ReadWriteLock;

public class ReadWorker extends Thread {

    private final ShareData shareData;

    public ReadWorker(ShareData shareData) {
        this.shareData = shareData;
    }

    @Override
    public void run() {
        try {
            while (true) {
                char[] readBuffer = shareData.read();

            }
        } finally {

        }
    }
}


package ReadWriteLock;

public class ShareData {

    private final char[] buffer;

    private final ReadWriteLock lock = new ReadWriteLock();

    public ShareData(int size) {
        this.buffer = new  char[size];
        for (int i = 0; i < size ; i++) {
           this.buffer[i] = '*';
        }
    }

    public char[] read() {
        try {
            lock.readLock();
            System.out.println(Thread.currentThread().getName() + " read  " + String.valueOf(buffer));
            return doRead();
        } finally {
            lock.readUnlock();
        }
    }

    private char[] doRead() {
        char[] newBuffer = new char[buffer.length];
        for (int i = 0; i < buffer.length; i++) {
            newBuffer[i] = buffer[i];
        }

        slowly(50);
        return newBuffer;
    }

    private void slowly(int i) {
        try {
            Thread.sleep(i);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public void write (char c) {
        try {
            lock.writeLock();
            System.out.println("write char is " + String.valueOf(c));
            doWrite(c);
        } finally {
            lock.writeUnLock();
        }
    }
    private void doWrite(char c) {
        for (int i = 0; i < buffer.length ; i++) {
             buffer[i] = c;
             slowly(i);
        }
    }
}


package ReadWriteLock;


public class ReadWriteLockClient {


    public static void main(String[] args) {
        final ShareData shareData = new ShareData(5);

        new ReadWorker(shareData).start();
        new ReadWorker(shareData).start();
        new ReadWorker(shareData).start();
        new ReadWorker(shareData).start();
        new ReadWorker(shareData).start();

        new WriteWorker(shareData,"qwer").start();
        new WriteWorker(shareData,"ASDF").start();
    }
}



```
