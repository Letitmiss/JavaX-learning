# stream

```
package com.cong.java8.stream;

import java.util.Arrays;
import java.util.List;

public class Dish {

    private final String name;
    //是否是素食
    private final boolean vegetarian;
    private final int calories;
    private final  Type type;

    public Dish(String name, boolean vegetarian, int calories, Type type) {
        this.name = name;
        this.vegetarian = vegetarian;
        this.calories = calories;
        this.type = type;
    }

    public String getName() {
        return name;
    }

    public boolean isVegetarian() {
        return vegetarian;
    }

    public int getCalories() {
        return calories;
    }

    public Type getType() {
        return type;
    }

    public enum Type {
        MEAT,FISH,OTHER
    }

    @Override
    public String toString() {
        return "Dish{" +
                "name='" + name + '\'' +
                '}';
    }

    public static final List<Dish> menu = Arrays.asList(
            new Dish("pork", false, 800, Dish.Type.MEAT),
            new Dish("pork", false, 800, Dish.Type.MEAT),
            new Dish("beef", false, 700, Dish.Type.MEAT),
            new Dish("chicken", false, 400, Dish.Type.MEAT),
            new Dish("french fries", true, 530, Dish.Type.OTHER),
            new Dish("rice", true, 350, Dish.Type.OTHER),
            new Dish("season fruit", true, 120, Dish.Type.OTHER),
            new Dish("pizza", true, 550, Dish.Type.OTHER),
            new Dish("prawns", false, 400, Dish.Type.FISH),
            new Dish("salmon", false, 450, Dish.Type.FISH)
    );

}


package com.cong.java8.stream;

import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;
import java.util.stream.Collectors;

/**
 * stream
 * 1. 简化代码,优化的API对流操作
 * 2. 并行操作,并行CPU执行
 * 3. 流是内部的
 */
public class SimpleStream {

    public static void main(String[] args) {
        List<String> dishNameByCollections = getDishNameByCollections(Dish.menu);
        System.out.println(dishNameByCollections);
        List<String> dishNameByStream = getDishNameByStream(Dish.menu);
        System.out.println(dishNameByStream);
    }

    /**
     * 传统实现: 选出低于400卡路里的事务,并且排序, 获取名字返回list
     * @param dishes
     * @return
     */
    private static List<String> getDishNameByCollections(List<Dish> dishes) {
        List<Dish> lowCalories = new ArrayList<>();
        //filter get calories less 400
        for (Dish dish : dishes) {
            if (dish.getCalories() < 400) {
                lowCalories.add(dish);
            }
        }

        //查看jvm线程 : 单线程执行
        try {
            Thread.sleep(100_000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        //sort
        Collections.sort(lowCalories, (d1,d2) -> Integer.compare(d1.getCalories(),d2.getCalories()));
        //get name
        List<String> dishName = new ArrayList<>();
        for (Dish lowCalory : lowCalories) {
            dishName.add(lowCalory.getName());
        }
        return dishName;
    }


    /**
     * stream实现: 选出低于400卡路里的事务,并且排序, 获取名字返回list
     *
     *   Collection新增的方法
     *
     *  default Stream<E> stream() {
     *   return StreamSupport.stream(spliterator(), false);
     *  }
     *
     *  Collectors的用法
     *
     * @param dishes
     * @return
     */
    private static List<String> getDishNameByStream(List<Dish> dishes) {
         return Dish.menu.stream().filter(dish -> dish.getCalories() < 400)
                 .sorted(Comparator.comparing(Dish::getCalories))
                 .map(Dish::getName)
                 .collect(Collectors.toList());

        /**
         *  parallelStream JVM会默认多线程执行,并发处理,
         *  Fork Join JDK7 引入
         *  ForkJoinPool.commonPool-worker-1
         *  ForkJoinPool.commonPool-worker-2
         *  ForkJoinPool.commonPool-worker-3
         */
       /* return Dish.menu.parallelStream().filter(dish -> {
                    try {
                        Thread.sleep(100_000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    return  dish.getCalories() < 400;
               })
                .sorted(Comparator.comparing(Dish::getCalories))
                .map(Dish::getName)
                .collect(Collectors.toList());*/
    }
}


package com.cong.java8.stream;

import com.sun.org.apache.xpath.internal.SourceTree;

import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.Stream;

/**
 * 流的特点
 * 1. 执行是内部的
 * 2. 流只能流向下一个点
 */
public class StreamDetail {

    public static void main(String[] args) {

        //特点, stream只能流向一个点,不能重复的流
       /* Dish.menu.stream().forEach(System.out::println);
        //这是两个新的流
        Dish.menu.stream().forEach(System.out::println);*/

        //流已经被操作不能重复操作
        Stream<Dish> stream = Dish.menu.stream();
        stream.forEach(System.out::println);

        // stream.forEach(System.out::println);
        //抛出 stream has already been operated upon or closed

        /**
         * /流的操作 Intermediate vs. terminal operations
         *
         * Intermediate  返回一个新的流的操作 filter map
         * Terminal operations 中断流操作
         */

        List<String> result = Dish.menu.stream().filter(d -> {
            System.out.println(" filtering ->" + d.getName());
            return d.getCalories() > 300;
        }).map(d -> {
            System.out.println(" map ->" + d.getName());
            return d.getName();
        }).limit(3).collect(Collectors.toList());

        /**
         * 执行过程,是一次filter,一次map, 而不是一次吧filter执行完,再执行map ; 而是单个元素过一个个关卡-最后流出
         *  filtering ->pork
         *   map ->pork
         * filtering ->pork
         *  map ->pork
         * filtering ->beef
         *  map ->beef
         */

        System.out.println("-----------2---------------");
        System.out.println(result);

        //Stream的方法操作理解

    }
}


```

