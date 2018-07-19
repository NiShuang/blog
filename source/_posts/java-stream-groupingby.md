---
title: Java8 stream 中利用 groupingBy 进行多字段分组求和
date: 2018-07-14 14:34:37
tags:
 - java 
 - 实习
categories: java笔记
---

## 从简单入手


Stream 作为 Java 8 的一大亮点，好比一个高级的迭代器（Iterator），单向，不可往复，数据只能遍历一次，遍历过一次后即用尽了，就好比流水从面前流过，一去不复返。

> [Java 8 中的 Streams API 详解](https://www.ibm.com/developerworks/cn/java/j-lo-java8streamapi/)

我们可以利用stream对数据进行分组并求和。示例如下：

``` java

List<String> items =
        Arrays.asList("apple", "apple", "banana",
                "apple", "orange", "banana", "papaya");

Map<String, Long> result =
        items.stream().collect(
                Collectors.groupingBy(
                        Function.identity(), Collectors.counting()
                )
        );

System.out.println(result);

```

输出如下：

```java
{papaya=1, orange=1, banana=2, apple=3}

```
<!-- more --> 

## 进阶需求

在实际需求中，我们可能需要对一组对象进行分组，而且分组的条件有多个。比如对国家和产品类型进行双重分组，一种比较复杂的方法是先对产品类型进行分组，然后对每一个产品类型中的国际名进行分组求和。示例如下：

```java
Map<String, List<item>> countMap = recordItems.stream().collect(Collectors.groupingBy(o -> o.getProductType()));

List<Record> records = new ArrayList<Record>;
countMap.keySet().forEach(productType -> {
	Map<String, Long> countMap1 = countMap.get(productType).stream().collect(Collectors.groupingBy(o -> o.getCountry(), Collectors.counting()));
	countMap1(key).stream().forEach(country -> {
		Record record = new Record();
	    record.set("device_type", productType);
	    record.set("location", country;
	    record.set("count", countMap1(key).intValue());
	    records.add(record);
	});
});
```


## 更好的解决方法

上面的方法在应对两个字段的分组要求时，还能应付的过来，但如果字段超过两个时，每增加一个字段，就会多出很多代码行，显然不太合理。更合理的方法是，增加一个 getKey()方法，返回多个字段组成的唯一key，比如通过下划线连接等等。示例如下：

```java

private static class RecordItem {
    private String deviceType;
    private String country;

    public RecordItem(String deviceType, String country) {
        this.deviceType = deviceType;
        this.country = country;
    }

    public String getKey() {
        return deviceType + "_" + country;
    }

    public static RecordItem parseKey(String s){
        String[] parts = s.split("_");
        return new RecordItem(parts[0], parts[1]);
    }
}


// 分组统计
Map<String, Long> countMap = recordItems.stream().collect(Collectors.groupingBy(o -> o.getKey(), Collectors.counting()));

List<Record> countRecords = countMap.keySet().stream().map(key -> {
    RecordItem recordItem = RecordItem.parseKey(key);
    Record record = new Record();
    record.set("device_type", recordItem.deviceType);
    record.set("location", recordItem.country;
    record.set("count", countMap.get(key).intValue());
    return record;
}).collect(Collectors.toList());
```

## 参考资料

 - [Java 8 中的 Streams API 详解](https://www.ibm.com/developerworks/cn/java/j-lo-java8streamapi/)
 - [Java 8 – 分组GroupBy](https://blog.csdn.net/u013078669/article/details/52717142)

---

> 文章标题：[Java8 stream 中利用g roupingBy 进行多字段分组求和](http://www.cielni.com/2018/07/14/java-stream-groupingby/)
> 文章作者：[Ciel Ni](http://www.cielni.com/about/)
> 文章链接：http://www.cielni.com/2018/07/14/java-stream-groupingby/
> 有问题或建议欢迎与我联系讨论，转载或引用希望标明出处，感激不尽！
