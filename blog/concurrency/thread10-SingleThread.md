# 单线程设计模式

```
package SingleThread;

/**
 * 单线程设计模式: 始终只能一个人通过,记录通过的信息
 *    多线程问题: 一个线程在执行,另一个线程修改了参数
 *
 *  共享资源问题: 有读 有写操作 会竞争锁, 还有共享资源操作
 *        解决办法: 修改方法和读取方法都加入一个锁 this锁, 这样执行没有问题,但是效率会有问题;
 */
public class Gate {

    private int counter = 0;
    private String name = "Nobody";
    private String address = "Nowhere";
    
    /**
     * 线程安全问题: 加锁解决
     */
    public synchronized void pass (String name, String address) {
        this.counter++;
        this.name = name;
        this.address = address;
        verify();
    }
    private void verify() {
        if(this.name.charAt(0) != this.address.charAt(0)) {
            System.out.println("******BROKEN*******" + toString());
        }
    }

    public synchronized String toString () {
        return "NO."+counter+":" + name+","+ address;
    }

}

package SingleThread;

public class Client {

    public static void main(String[] args) {
        Gate gate = new Gate();
        UserThread user = new UserThread("Baobao", "Beijing", gate);
        UserThread user1 = new UserThread("Shangbao", "Shanghai", gate);
        UserThread user2 = new UserThread("Guangbao", "Guangzhou", gate);

        user.start();
        user1.start();
        user2.start();

    }
}

```



