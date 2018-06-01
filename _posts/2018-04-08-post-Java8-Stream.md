---
layout:     post
title:      "Java8-Stream"
subtitle:   ""
date:       2018-04-08 12:00:00
author:     "闫祥"
header-img: "img/todo/Stream.png"
tags:       Java8
---

# Stream

## 流定义（java.util.stram.Stream）
> 从支持数据处理操作的源生成的元素序列
> 流就像一个延迟创建的集合：只有在消费者需要的时候才会计算值
> 流面向的是计算

``` java
//例如在Java Doc 里的示例
int sum = widgets.stream()
                  .filter(w -> w.getColor() == RED)
                  .mapToInt(w -> w.getWeight())
                  .sum();
```
### 流操作
- 中间操作： filter mapToInt map limit 等可以连成一条流水线
- 终端操作： collect sum 触发流水线执行并关闭它

流的使用一般包括三件事：
- 一个数据源（如集合）来执行一个查询；
- 一个中间操作链，形成一条流的流水线；
- 一个终端操作，执行流水线，并能生成结果。

Stream API提供的操作：  

- 中间操作  

	|操作|类型|返回类型|操作类型|函数描述符|
	|---|---|-------|------|----------|
	|filter   | 中间  |  Stream<T> |  Predicate<T> | T -> boolean |
	|map   | 中间  |  Stream<T> |  Function<T, R> | T -> R |
	|limit   | 中间  |  Stream<T> |   |   |
	|sorted   | 中间  |  Stream<T> |  Comparator<T> | (T,) -> int |
	|distinct   | 中间  |  Stream<T>   |   |   |  

- 终端操作  

	|操作|类型|目的|
	|---|---|---|
	| forEach | 终端 | 消费流中的每个元素并对其应用Lambda。这一操作返回void  |
	|  count | 终端  |  返回流中元素的个数，这一操作返回long |
	| collect | 终端 | 把流归结成一个集合 |



> 参考资料：
> [Java 8实战](https://www.amazon.cn/mn/detailApp/ref=asc_df_B01M9GP6JA2927275/?asin=B01M9GP6JA&tag=douban_kindle-23&creative=2384&creativeASIN=B01M9GP6JA&linkCode=df0)

*****
[记录并分享自己的学习与成长](http://cbrothercoder.com/)
