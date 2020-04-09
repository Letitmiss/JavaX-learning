# stream2


```
package com.cong.java8.stream;

import com.sun.org.apache.xerces.internal.impl.xpath.regex.Match;

import java.io.IOException;
import java.lang.reflect.Array;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.*;
import java.util.function.Supplier;
import java.util.stream.DoubleStream;
import java.util.stream.IntStream;
import java.util.stream.Stream;

/**
 * 创建 stream
 */
public class CreateStream {

    public static void main(String[] args) {

        //通过集合创建 stream;  parallelStream()  stream()
        Dish.menu.stream().forEach(System.out::println);

        //通过Stream.of
        Stream.of(Dish.menu).forEach(System.out::println);
        Stream.of("Hello","world","haha","hehe").forEach(System.out::println);

        //通过数组Arrays.stream()
        String[] array= {"123","456","tom","world"};
        Arrays.stream(array).forEach(System.out::println);

        //通过file Arrays.stream()
        System.out.println("=======33333===========");
        Path path = Paths.get("E:\\JAVA-High\\java8\\src\\main\\java\\com\\cong\\java8\\stream\\data.txt");
        try {
            Stream<String> lines = Files.lines(path);
            lines.forEach(System.out::println);
        } catch (IOException e) {
            e.printStackTrace();
            throw new RuntimeException(e);
        }

        //通过 iterator创建
        System.out.println("=======4444===========");
        Stream<Integer> iterate = Stream.iterate(0, n -> n + 2);
        //创建出来的流是无限的
        //iterate.forEach(System.out::println);
        //limit取出来前100
        iterate.limit(10).forEach(System.out::println);

        System.out.println("=======5555===========");
        //通过generate
        Stream<Double> generate = Stream.generate(Math::random);
        //这个流也是无限的
        generate.limit(10).forEach(System.out::println);

        //IntStream 的静态方法 generate iterate range等
        //DoubleStream 等

        //Stream<T> generate(Supplier<T> s)
        //自定义对象,Supplier<T> 可以返回无限多个对象
        //Objects.requireNonNull(s);
        System.out.println("=======6666===========");
        Stream<Obj> generate1 = Stream.generate(new ObjSupplier());
        generate1.forEach(System.out::println);


    }


    public static class ObjSupplier implements Supplier<Obj> {

        private int index = 0 ;

        private Random random = new Random(System.currentTimeMillis());
        @Override
        public Obj get() {
            index = random.nextInt(100);
            return new Obj(index,"name->"+index);
        }
    }


    public static class Obj {
        private int id;
        private String name;

        public Obj(int id, String name) {
            this.id = id;
            this.name = name;
        }

        public int getId() {
            return id;
        }

        public String getName() {
            return name;
        }

        @Override
        public String toString() {
            return "Obj{" +
                    "id=" + id +
                    ", name='" + name + '\'' +
                    '}';
        }
    }
}



```
