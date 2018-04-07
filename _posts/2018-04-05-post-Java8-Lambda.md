---
layout:     post
title:      "Java8-Lambda"
subtitle:   ""
date:       2018-04-04 12:00:00
author:     "闫祥"
header-img: "img/todo/lambda.png"
tags:       Java lambda Java8
---

# lambda

<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [lambda](#lambda)
		- [lambda优势](#lambda优势)
		- [lambda表达式](#lambda表达式)
		- [函数式接口](#函数式接口)
		- [类型检查，类型推断](#类型检查类型推断)
		- [方法引用](#方法引用)
		- [lambda表达式于方法引用的复合](#lambda表达式于方法引用的复合)

<!-- /TOC -->

### lambda优势
  替代匿名内部类或者定义的接口实现类传参，不用再把代码封装成对象传参
    ```Java

    class FilterParcel {
    public List<Parcel> filter(List<Parcel> parcels, CheckPackageStrategy strategy) {
        List<Parcel> parcelList = new ArrayList<>();
        for (Parcel parcel : parcels) {
            if (strategy.test(parcel)) {
                parcelList.add(parcel);
            }
        }
        return parcelList;
      }
    }
    class Parcel {
      Double weight;
      Long storeId;
    }
    interface CheckPackageStrategy {
      boolean test(Parcel parcel);
    }

    // 使用匿名内部类
    public void useAnonymousClass() {
        List<Parcel> parcels = new ArrayList<>();
        List<Parcel> filter = new FilterParcel().filter(parcels, new CheckParcelStrategy() {
            @Override
            public boolean test(Parcel parcel) {
                return parcel != null && parcel.storeId != null && parcel.storeId.equals(110L);
            }
        });
    }

    //传递接口实现类
    class CheckWeightPackageStrategy implements CheckPackageStrategy {
      @Override
      public boolean test(Parcel parcel) {
        return parcel != null && parcel.weight != null && parcel.weight > 2.1;
      }
    }
    ```

    ```Java
    //使用泛型和lambda表达式


        @Test
        public void useLam() {
            List<Parcel> parcels = new ArrayList<>();
            new Filter().filter(parcels, (Parcel p) -> p.storeId.equals(110L));

            List<Integer> integers = new ArrayList<>();
            new Filter().filter(integers, (Integer i) -> i % 2 == 0);
        }

        class Filter {

            public <T> List<T> filter(List<T> list, CheckStrategy<T> strategy) {
                List<T> tList = new ArrayList<>();
                for (T t : list) {
                    if (strategy.test(t)) {
                        tList.add(t);
                    }
                }
                return tList;
            }

        }
        @FunctionalInterface
        interface CheckStrategy<T> {
            boolean test(T t);
        }

        class FilterParcel {

            public List<Parcel> filter(List<Parcel> parcels, CheckParcelStrategy strategy) {
                List<Parcel> parcelList = new ArrayList<>();
                for (Parcel parcel : parcels) {
                    if (strategy.test(parcel)) {
                        parcelList.add(parcel);
                    }
                }
                return parcelList;
            }

        }

    /*
     * 理论上来说，你在Java 8之前做不了的事情，lambda也做不了。
     * 使用Lambda后就用不着再用匿名类写一堆笨重的代码，体验行为参数化的好处了。
     */

    ```
### lambda表达式
Lambda表达式基本语法：

> (parameters) -> expression
> 或（请注意语句的花括号）
> (parameters) -> { statements; }

Lambda表达式组成有三个部分

1. 参数列表——即() 和其中的参数。

2. 箭头——箭头->把参数列表与Lambda主体分隔开。

3. Lambda主体——要传递的代码，表达式就是Lambda的返回值了。

Java8中有效的lambda表达式:
```Java
(String s) -> s.length() //第一个Lambda表达式具有一个String类型的参数并返回一个int。Lambda没有return语句，因为已经隐含了return
(Parcel a) -> a.getWeight() > 150 //第二个Lambda表达式有一个Parcel类型的参数并返回一个boolean（Parcel的重量是否超过150克）
(int x, int y) -> {
    System.out.println("Result:");
    System.out.println(x + y);
    // 第三个Lambda表达式具有两个int类型的参数而没有返回值（void返回）。注意Lambda表达式可以包含多行语句，这里是两行
}
() -> 42 //第四个Lambda表达式没有参数， 返回一个int
(Parcel a1, Parcel a2) -> a1.getWeight().compareTo(a2.getWeight()) //第五个Lambda表达式具有两个Parcel类型的参数，返回一个int：比较两个Apple的重量
```
### 函数式接口
“函数式接口就是只定义一个抽象方法的接口”

新的Java API，函数式接口都带有@FunctionalInterface的注解。如果带有该注解，但该接口并不是函数式接口，编译器在编译的时候会报错。

在Java 8 java.util.function包中引入了几个新的函数式接口：
- java.util.function.Predicate<T>: 有一个test() 方法，接受泛型<T>对象，并返回一个boolean。
- java.util.function.Consumer<T>: 有一个accept() 方法，接受泛型<T>对象，没有返回值。
- java.util.function.Function<T, R>: 有一个accept() 方法，接受泛型<T>对象，返回泛型<R>对象。

特殊类型专用函数式接口：
- IntPredicate: 免去频繁的装箱，类似的还有LongPredicate

函数式接口汇总表：
函数式接口 | 函数描述符  | 原始类型特化
---|---|---
Predicate<T>|T->boolean|IntPredicate,LongPredicate,DoublePredicate
Consumer<T>|T->void|IntConsumer,LongConsumer, DoubleConsumer
Function<T,R> | T->R | IntFunction<R>,IntToDoubleFunction, IntToLongFunction,LongFunction<R>,LongToDoubleFunction,LongToIntFunction,DoubleFunction<R>,ToIntFunction<T>,ToDoubleFunction<T>,ToLongFunction<T>
Supplier<T>|()->T|BooleanSupplier,IntSupplier,LongSupplier, DoubleSupplier
UnaryOperator<T>|T->T|IntUnaryOperator,LongUnaryOperator,DoubleUnaryOperator
BinaryOperator<T>|(T,T)->T|IntBinaryOperator,LongBinaryOperator,DoubleBinaryOperator
BiPredicate<L,R>|(L,R)->boolean|
BiConsumer<T,U>|(T,U)->void|ObjIntConsumer<T>,ObjLongConsumer<T>,ObjDoubleConsumer<T>
BiFunction<T,U,R>|(T,U)->R|ToIntBiFunction<T,U>,ToLongBiFunction<T,U>,ToDoubleBiFunction<T,U>

注意：任何函数式接口都不允许抛出受检异常（checked exception）。如果你需要Lambda表达式来抛出异常，有两种办法：定义一个自己的函数式接口，并声明受检异常，或者把Lambda包在一个try/catch块中

### 类型检查，类型推断
Lambda的类型是从lambda的上下文推断出来的。同时Java编译器会从上下文推断出用什么函数式接口来配合lambda表达式，也就是编译器可以推断出lambda的签名。

局部变量的使用：lambda表达式可以使用的局部变量必须是 (final）修饰的。其原因如下：
1. 实例变量和局部变量背后的实现有一个关键不同。实例变量都存储在堆中，而局部变量则保存在栈上。如果Lambda可以直接访问局部变量，而且Lambda是在一个线程中使用的，则使用Lambda的线程，可能会在分配该变量的线程将这个变量收回之后，去访问该变量。因此，Java在访问自由局部变量时，实际上是在访问它的副本，而不是访问原始变量。如果局部变量仅仅赋值一次那就没有什么区别了——因此就有了这个限制。
2. 这一限制不鼓励你使用改变外部变量的典型命令式编程模式

### 方法引用
方法引用可以被看作仅仅调用特定方法的Lambda的一种快捷写法。它的基本思想是，如果一个Lambda代表的只是“直接调用这个方法”，那最好还是用名称来调用它，而不是去描述如何调用它。事实上，方法引用就是让你根据已有的方法实现来创建Lambda表达式。但是，显式地指明方法的名称，代码的可读性会更好。
> 当你需要使用方法引用时，目标引用放在分隔符::前，方法的名称放在后面. (System.out::println)

方法引用主要有三类：
1. 指向静态方法的方法引用（例如Integer的parseInt方法，写作Integer::parseInt）
2. 指向任意类型实例方法的方法引用（例如String的length方法，写作String::length）
3. 指向现有对象的实例方法的方法引用（假设你有一个局部变量expensiveTransaction用于存放Transaction类型的对象，它支持实例方法getValue，那么你就可以写expensiveTransaction::getValue）

构造函数引用
> 对于一个现有构造函数，你可以利用它的名称和关键字new来创建它的一个引用 ClassName::new
```Java
Supplier<Parcel> c1 = Parcel::new;    //构造函数引用指向默认的Apple()构造函数
Parcel a1 = c1.get();    //调用Supplier的get方法将产生一个新的Apple”
// 等价于
Supplier<Parcel> c1 = () -> new Parcel();   //利用默认构造函数创建Apple的Lambda表达式
Parcel a1 = c1.get();   //调用Supplier的get方法将产生一个新的Apple


Function<Integer, Parcel> c2 = Parcel::new;  //指向Parcel(Integer weight)的构造函数引用
Parcel a2 = c2.apply(110);  //调用该Function函数的apply方法，并给出要求的重量，将产生一个Parcel
//等价于
Function<Integer, Parcel> c2 = (weight) -> new Parcel(weight); //用要求的重量创建一个Parcel的Lambda表达式
Parcel a2 = c2.apply(110); //调用该Function函数的apply方法，并给出要求的重量，将产生一个新的Apple对象

```
### lambda表达式于方法引用的复合

```Java

    @Test
    public void recombination() {
        //比较器复合
        List<Parcel> parcels = new ArrayList<>();
        Comparator<Parcel> c = Comparator.comparing(Parcel::getWeight);
        parcels.sort(Comparator.comparing(Parcel::getWeight).reversed());//逆序
        parcels.sort(Comparator.comparing(Parcel::getWeight).reversed().thenComparing(Parcel::getStoreId));//比较器链 进一步进行比较

        //谓词复合
        //谓词接口包含 negate and 和 or
        Predicate<Parcel> weightParcel = (Parcel p) -> p.weight > 5;

        Predicate<Parcel> negateWeightParcel = weightParcel.negate();
        Predicate<Parcel> weightAndParcel = weightParcel.and(p -> p.storeId == 110L);
        Predicate<Parcel> weightAndOrParcel = weightParcel.and(p -> p.storeId == 110L).or(parcel -> parcel.totalItem > 2);

    }

```


> 参考资料：
> [Java 8实战](https://www.amazon.cn/mn/detailApp/ref=asc_df_B01M9GP6JA2927275/?asin=B01M9GP6JA&tag=douban_kindle-23&creative=2384&creativeASIN=B01M9GP6JA&linkCode=df0)

*****
[NoCopy 记录并分享自己的学习成长过程](http://cbrothercoder.com/)
