# 不可变对象
```
1）所有成员变量必须是private

2）最好同时用final修饰(非必须)

3）不提供能够修改原有对象状态的方法

   最常见的方式是不提供setter方法

   如果提供修改方法，需要新创建一个对象，并在新创建的对象上进行修改

4）通过构造器初始化所有成员变量，引用类型的成员变量必须进行深拷贝(deep copy)

5）getter方法不能对外泄露this引用以及成员变量的引用

6）最好不允许类被继承(非必须)
```

## 不可变对象,不可变类是线程安全的

```
package immutable;

import java.util.Collections;
import java.util.List;

/**
 *  线程安全的类: 不可变对象 String Thread
 *
 */
final public class Person {

    private final String name;

    private final String address;

    private final List<String> list;

    public Person(String name, String address,List list) {
        this.name = name;
        this.address = address;
        this.list = list;
    }

    public String getName() {
        return name;
    }

    public String getAddress() {
        return address;
    }

    //这个list的返回可以被修改,这里要返回不可变对象
    public List<String> getList() {
        //return list;
        return Collections.unmodifiableList(list);
    }

    @Override
    public String toString() {
        return "person{" +
                "name='" + name + '\'' +
                ", address='" + address + '\'' +
                '}';
    }
}

package immutable;

public class UserPersonThread extends Thread {

    private Person person;

    public UserPersonThread(Person person) {
        this.person = person;
    }

    @Override
    public void run() {
        while (true) {
            System.out.println(Thread.currentThread().getName() +
                    " print " + person.getName() + ", " + person.getAddress());
        }
    }
}

package immutable;

import java.util.Arrays;
import java.util.stream.IntStream;

/**
 * 不可变对象:
 * 1. 不可变对象一定是线程安全的 String
 * 2. 可变对象不一定是线程不安全的 StringBuffer 线程安全的  StringBuilder线程不安全的;
 *
 */
public class ImmutableTestClient {

    public static void main(String[] args) {

        //share data
        Person person = new Person("Alex", "Gansu", Arrays.asList(1,2));

        IntStream.rangeClosed(0,5).forEach(i -> {
            new UserPersonThread(person).start();
            }
        );
    }
}

```
# 不可变对象,多线程测试
```
package immutable;


/**
 *  不可变对象的性能测试
 *
 *  asynchronous 异步
 *  synchronous  同步
 */
public class ImmutablePerformance {

    public static void main(String[] args) throws InterruptedException {
        long start = System.currentTimeMillis();
        SyncObj syncObj = new SyncObj();
        syncObj.setName("Alex");

        ImmubaleObj tom = new ImmubaleObj("Tom");

        Thread thread = new Thread(() -> {
            for (int i = 0; i < 1_000_000; i++) {
                System.out.println(Thread.currentThread().getName() + "," + syncObj.toString());
            }
        });
        thread.start();

        Thread thread2 = new Thread(() -> {
            for (int i = 0; i < 1_000_000; i++) {
                System.out.println(Thread.currentThread().getName() + "," + syncObj.toString());
            }
        });
        thread2.start();
        thread.join();
        thread2.join();

        long end = System.currentTimeMillis();
        System.out.println("time : " + (end-start));
    }

}

final  class ImmubaleObj {
    private final String name;

    public ImmubaleObj(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "ImmubaleObj{" +
                "name='" + name + '\'' +
                '}';
    }
}

class SyncObj {
    private String name;

    public String getName() {
        return name;
    }

    public synchronized void setName(String name) {
        this.name = name;
    }

    @Override
    public synchronized String toString() {
        return "SyncObj{" +
                "name='" + name + '\'' +
                '}';
    }
}
```


