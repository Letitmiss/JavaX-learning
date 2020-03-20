```
package com.cong.concurrency;

import javax.sound.midi.Soundbank;
import java.util.PrimitiveIterator;

public class Concurrency05 {

    //方法区
    private  int i = 0;

    //引用地址存储到方法区
    private byte[] bytes = new byte[1024];

    private static int counter = 0 ;

    //JVM will creat thread main
    public static void main(String[] args) {
        //create a JVM stack
     /*   try {
            add(0);
        }catch (Error e) {
            e.printStackTrace();
            // StackOverflowError,死循环栈溢出异常,
            System.out.println(counter);
        }*/

        /**
         *  Thread(ThreadGroup group, Runnable target, String name, long stackSize)
         *
         *   指定了一个较高的值stackSize参数可以允许抛出一个前一个线程来实现更大的递归深度StackOverflowError
         *   main函数的stackSize是JVM创建的, 需要自己创建一个线程测试
         * 分配一个新的 Thread对象，以便它具有 target作为其运行对象，将指定的 name正如其名，以及属于该线程组由称作 group ，并具有指定的 堆栈大小
         */
         new Thread(null,new Runnable() {
             @Override
             public void run() {
                 try {
                     add(1);
                 }catch (Error e) {
                     e.printStackTrace();
                     System.out.println(counter);
                 }
             }

             private void add(int i) {
                 ++counter;
                 add(i+1);
             }
         },"Test",1 << 24).start();

        /**
         * stack参数代表线程栈的大小, 如果没有指定的默认是0, 0 表示该平台会忽略该参数, 该参数被JNI去加载 -Xxxm10M
         *
         * http://www.matools.com/api/java8
         */

    }


}

```
package com.cong.concurrency;

public class Concurrency06 {

    private static int counter = 1;

    public static void main(String[] args) {

        try {
            for (int i = 0; i < Integer.MAX_VALUE ; i++) {
                counter++;
                new Thread( () -> {
                    while (true) {
                        try {
                            Thread.sleep(1);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }).start();
            }
        }catch (Error error) {
            error.printStackTrace();
            System.out.println(counter);
        }
    }
}


```

```
