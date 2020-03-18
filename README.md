# JavaX-learning
javaX-learning



# Java Encryption Algorithm

1.[加密介绍](https://github.com/Letitmiss/JavaX-learning/blob/master/blog/Encryption/01.Encryption.md)

# CDN 简介

# retrofit2

# [okhttp3](https://github.com/Letitmiss/JavaX-learning/blob/master/blog/okhttp/1.okhttp.md)

# rxjava

# 线程池的创建与使用

# SpringBoot


# 部署

# [Docker](https://github.com/Letitmiss/JavaX-learning/blob/master/blog/docker/1.docker.md)



# java8的新特性学习 
https://www.cnblogs.com/xingzc/p/5778090.html optional

https://www.cnblogs.com/puyangsky/p/7608741.html 异步分析理解
 
https://blog.csdn.net/u014082714/article/details/51423756   学习

https://blog.csdn.net/Hatsune_Miku_/article/details/73435580 转化map的使用

https://www.cnblogs.com/CarpenterLee/p/6550212.html

https://blog.csdn.net/IO_Field/article/details/54971679

https://www.jianshu.com/p/82ed16613072

# 谷歌
https://plus.google.com/communities/104092405342699579599

# 美国ID账号

https://bbs.feng.com/thread-htm-fid-365.html

美区ID一:（可安装小火箭 Quantumult FastSocks ）
帐号	yeswallios1@yeah.net
密码	Yeswall5566778899

美区ID二:（可安装 小火箭 Potatso 2 FastSocks ）
帐号	yeswalliosthe2@yeah.net
密码	Yeswall112233

逗比根据地
https://doubibackup.com/


```
package com.cong.concurrency;

/**
 * java8 的借口注释
 */

@FunctionalInterface
public interface CalculatorStrategy {

    double calculator(double salary,double bonus);
}


package com.cong.concurrency;

public class CalculatorStrategyImpl implements CalculatorStrategy {

    @Override
    public double calculator(double salary, double bonus) {
        return salary*0.10 + bonus * 0.15;
    }
}


package com.cong.concurrency;

import sun.reflect.generics.tree.VoidDescriptor;

import javax.sound.midi.Soundbank;

/**
 * 创建线程
 * 文档 http://www.matools.com/api/java8
 * jvm的启动有, main线程启动(非守护线程), 还有好几个守护线程启动jconsole可以看到
 * 创建线程
 * 1. Thread类, start才表示启动这个线程
 * 2. 线程状态-runnable--running--blocked
 *
 * java应用程序的main函数是一个线程, jvm启动就会启动main, 线程名字就是main
 * 实现一个线程,创建Thread类的实例,override run方法, 并且调用start方法
 * 在JVM启动之后,实际上启动了多个线程,但是至少有一个非守护线程
 * 调用start方法,此时至少有两个线程, 一个是启动start线程, 还有一个是被启动的start线程
 * 线程的生命周期,分为new runnable running block terminate
 */
public class Concurrency01 {

    public static void main(String[] args) {
        
        
/*       new Thread("READ-Thread") {

          @Override
          public void  run() {
              readFromDatabase();
          }
       }.start();


      new Thread("WRITE-Thread") {
          @Override
          public void run () {
              writeDataFile();
          }
      }.start();*/

      new Thread(new Runnable() {
            public void run() {
                readFromDatabase();
            }
        }).start();

        new Thread(new Runnable() {
            public void run() {
                writeDataFile();
            }
        }).start();



        /*readFromDatabase();
        writeDataFile();*/
    }

    public static void readFromDatabase() {
        //read data from database
        try {
            println("Begin read data from database");
            System.out.println(Thread.currentThread().getName());
            Thread.sleep(1000*20L);
            println("Read data done and start handle it.");

        } catch (InterruptedException e) {
            e.printStackTrace();
        }

    }

    private static void writeDataFile() {

        try {
            println("Begin write data to file");
            System.out.println(Thread.currentThread().getName());
            Thread.sleep(1000* 10L);
            println("Write  data to file done and start handle it. ");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    private static void println(String message) {
        System.out.println(message);
    }


}



package com.cong.concurrency;

import javax.sound.midi.Soundbank;

/**
 * 创建线程 runnable接口
 * 需求: 排队问题,多个窗口叫号码
 * 1. runnable只是Thread的一个接口, 提供了一个run方法实现的独立接口
 *
 */
public class Concurrency02 {


    private final static int Max = 1000;

    public static void main(String[] args) {
        /**
         * V1: 共享数据使用static,三个线程使用同一个数据, static周期比较长,
         */
        new TicketWindowThread("1号柜台").start();
        new TicketWindowThread("2号柜台").start();
        new TicketWindowThread("3号柜台").start();


        /**
         * 使用一个runnable实现了,多个线程之间的使用同一份数据 一个实例的管理中心 , 将可执行单元和线程控制分离开来
         */
        final TicketWindowRunnable ticketWindow = new TicketWindowRunnable();
        Thread thread1 = new Thread(ticketWindow, "1号");
        Thread thread2 = new Thread(ticketWindow, "2号");
        Thread thread3 = new Thread(ticketWindow, "3号");
        thread1.start();
        thread2.start();
        thread3.start();


        /**
         * java 8 创建线程
         */
        final Runnable ticket = () -> {
             int index = 1 ;
             while (index <= Max) {
                 try {
                     System.out.println(Thread.currentThread().getName()+",current num : " + index);
                     index++;
                     Thread.sleep(1000*1L);
                 } catch (InterruptedException e) {
                     e.printStackTrace();
                 }
             }
        };
        Thread ticket1 = new Thread(ticket, "1号");

        /**
         * 结合写法,命名参数设置,启动一起执行
         */

        new Thread(() -> {
            for (int i = 0; i < 10 ; i++) {
                System.out.println(Thread.currentThread().getName()+":"+i);
            }
        },"java8简洁写法").start();

    }

}



package com.cong.concurrency;

/**
 * Thread API介绍
 * 1. 构造函数,细节 threadgroup  stacksize 线程与线程组,线程与栈的关系
 * 2. 静态方法和非静态方法
 * 3.
 */
public class Concurrency03 {


    public static void main(String[] args) {

        /**
         * 构造函数
         * 1. Thread() 创建新的对象
         * 创建线程对象,默认线程名, Thread-0 开始计数
         */
        Thread thread1 = new Thread();
        thread1.start();
    }
}


package com.cong.concurrency;

/**
 * runnable的设计模式
 */
public class TaxCalculator {

    private final double salary;

    private final double bonus;

    private CalculatorStrategy calculatorStrategy;

    public TaxCalculator(double salary,double bonus,CalculatorStrategy calculatorStrategy) {
        this.salary = salary;
        this.bonus = bonus;
    }

    protected double calTax() {
        return calculatorStrategy.calculator(salary,bonus);
    }

    public double calculator() {
        return this.calTax();
    }


    public double getSalary() {
        return salary;
    }

    public double getBonus() {
        return bonus;
    }

    public CalculatorStrategy getCalculatorStrategy() {
        return calculatorStrategy;
    }

    public void setCalculatorStrategy(CalculatorStrategy calculatorStrategy) {
        this.calculatorStrategy = calculatorStrategy;
    }
}


package com.cong.concurrency;

/**
 * runnable的设计模式
 */
public class TaxCalculator {

    private final double salary;

    private final double bonus;

    private CalculatorStrategy calculatorStrategy;

    public TaxCalculator(double salary,double bonus,CalculatorStrategy calculatorStrategy) {
        this.salary = salary;
        this.bonus = bonus;
    }

    protected double calTax() {
        return calculatorStrategy.calculator(salary,bonus);
    }

    public double calculator() {
        return this.calTax();
    }


    public double getSalary() {
        return salary;
    }

    public double getBonus() {
        return bonus;
    }

    public CalculatorStrategy getCalculatorStrategy() {
        return calculatorStrategy;
    }

    public void setCalculatorStrategy(CalculatorStrategy calculatorStrategy) {
        this.calculatorStrategy = calculatorStrategy;
    }
}


package com.cong.concurrency;

/**
 * 模板方法
 */
public abstract class TemplateMethod {

    /**
     * 算法定义好了,之后使用final子类就无法复写里面的流程, 但是可以自由实现里面的部分细节
     * @param message
     */
    public final void println(String message) {

        System.out.println("##############");
        wrapPrint(message);
        System.out.println("-------------------");
    }

    protected  void wrapPrint(String message){
        System.out.println("public  : " + message);
    }

    public static void main(String[] args) {

        new TemplateMethod() {
            @Override
            protected void wrapPrint(String message) {
                System.out.println("custome :" + message);
            }
         }.println("Hello");
    }
}



package com.cong.concurrency;

public class Thread01 {

    public static void main(String[] args) {

        new Thread("Thread_READ") {
            @Override
            public void run() {
                readData();
            }
        }.start();

        new Thread("Thread_WRITE") {
            @Override
            public void run () {
                writeDataToFile();
            }

        }.start();


    }


    public static void readData() {

        try {
            println(Thread.currentThread()
                    .getName()+ ":"+ "Begin read data from database");
            Thread.sleep(1000*20L);

            println(Thread.currentThread()
                    .getName()+ ":"+ "End read data from database");

        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public  static  void  writeDataToFile() {
        try {
            println(Thread.currentThread().getName() + ":" + "Begin write data to file");
            Thread.sleep(1000*10L);
            println(Thread.currentThread().getName() + ":" + "End  write data to file");

        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void println(String message) {
        System.out.println(message);
    }

}


package com.cong.concurrency;


/**
 * 使用一个runnable实现了,多个线程之间的使用同一份数据 一个实例的管理中心 , 将可执行单元和线程控制分离开来
 */
public class TicketWindowRunnable implements Runnable {

    private int index = 1 ;
    private final int Max = 1000;

    @Override
    public void run() {
        while (index <= Max) {

            try {
                System.out.println(Thread.currentThread().getName()+",current num : " + index);
                index++;
                Thread.sleep(1000*1L);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}


package com.cong.concurrency;

public class TicketWindowThread extends Thread {

    private String windowName;

    private final int MAX = 500 ;
    /**
     * 没有static时候,Index是互相独立的,有static 是属于整个类的,三个线程使用同一份数据
     * static是随着jvm加载,类销毁了, static都不一定销毁, 存储不在类里面
     */
    private static int INDEX = 1;

    public TicketWindowThread(String windowName) {
        this.windowName = windowName;
    }

    @Override
    public void run() {

        while (INDEX <= MAX) {
            System.out.println(windowName + ",current num : " + INDEX);
            INDEX++;
        }
    }
}


```
