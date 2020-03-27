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
```
