下面举个例子 `Java8` 新语法实现思路的多种解法
```
/**
 *
 * 1. 计数
 * Collectors.counting
 * count
 * 2. 最值
 * Collectors.maxBy
 * 3. 求和
 * Collectors.summingInt
 * 4. 求平均值
 * Collectors.averagingInt
 * 5. 连接字符串
 * Collectors.joining
 * 6. 一般归约
 * Collectors.reducing
 * 7. 转换
 * Collectors.toCollection / Collectors.collectingAndThen
 * 8. 数值流
 * IntStream、DoubleStream、LongStream
 * 9. 映射
 * Collectors.toMap
 * Function.identity()
 * 10. 数据分区
 * Collectors.partitioningBy
 * 11. 分组
 * Collectors.groupingBy
```

```
public class Java8 {

    static List<People> list = Lists.newArrayList(
            new People(1L, "周杰伦", "男", 30, 12),
            new People(2L, "汪峰", "男", 30, 10),
            new People(3L, "刘亦菲", "女", 20, 15),
            new People(4L, "蔡依林", "女", 18, 18),
            new People(5L, "张杰", "男", 30, 20),
            new People(6L, "薛之谦", "男", 28, 30)
    );

    /**
     * 求集合薪资总和
     */
    @Test
    public void example1() {
        list.stream().map(People::getSalary).reduce((s1, s2) -> s1 + s2).ifPresent(System.out::println);
        Optional<Double> opDouble = list.stream().map(People::getSalary).collect(Collectors.reducing((s1, s2) -> s1 + s2));
        opDouble.ifPresent(System.out::println);
        Number number = list.stream().collect(Collectors.reducing(0,People::getSalary,(s1,s2) -> s1.doubleValue() + s2.doubleValue()));
        System.out.println(number.doubleValue());
        System.out.println(list.stream().collect(Collectors.summingDouble(People::getSalary)));
        System.out.println(list.stream().collect(Collectors.summarizingDouble(People::getSalary)).getSum());
        System.out.println(list.stream().mapToDouble(People::getSalary).sum());
    }

    /**
     * 求集合薪资最大 (最小)
     */
    @Test
    public void example2() {
        list.stream().collect(Collectors.maxBy(Comparator.comparing(People::getSalary))).ifPresent(people -> System.out.println(people.getSalary()));
        list.stream().map(People::getSalary).reduce(BinaryOperator.maxBy(Double::compareTo)).ifPresent(System.out::println); // minBy()
        list.stream().map(People::getSalary).reduce((s1,s2) -> s1 > s2 ? s1 : s2).ifPresent(System.out::println); // <
        System.out.println(list.stream().map(People::getSalary).reduce(0d,Double::max));
        Map<String,Optional<People>> mapOp = list.stream().collect(Collectors.groupingBy(People::getSex,Collectors.reducing((p1,p2) -> p1.getSalary() > p2.getSalary() ? p1 : p2)));
        mapOp.get("男").ifPresent(people -> System.out.println(people.getSalary()));
        list.stream().map(People::getSalary).collect(Collectors.reducing((s1,s2) -> s1 > s2 ? s1 : s2)).ifPresent(System.out::println);
        list.stream().max(Comparator.comparing(People::getSalary,Double::compare)).ifPresent(p -> System.out.println(p.getSalary()));
        list.stream().max(Comparator.comparing(People::getSalary)).ifPresent(p -> System.out.println(p.getSalary()));
        list.stream().map(People::getSalary).max(Double::compareTo).ifPresent(System.out::println); // min
        System.out.println(list.stream().collect(Collectors.summarizingDouble(People::getSalary)).getMax()); //getMin()

        list.stream().sorted(Comparator.comparing(People::getSalary).reversed()).map(People::getSalary).findFirst().ifPresent(System.out::println); // reversed()
        list.stream().map(People::getSalary).sorted(Comparator.reverseOrder()).findFirst().ifPresent(System.out::println); // Comparator.naturalOrder()
        list.stream().map(People::getSalary).sorted(Comparator.comparingDouble(Double::doubleValue).reversed()).findFirst().ifPresent(System.out::println);

        List<Double> doubles = list.stream().filter(Objects::nonNull).map(People::getSalary).collect(Collectors.toList());
        Collections.sort(doubles, Double::compareTo);
        System.out.println(doubles.get(doubles.size() - 1)); // get(0)
        Collections.reverse(doubles);
        doubles.stream().findFirst().ifPresent(System.out::println);

        Map<String, People> map = list.stream().sorted(Comparator.comparing(People::getSalary, Double::compareTo))
                .collect(Collectors.toMap(People::getSex, Function.identity(), (p1, p2) -> p2));
        double d = map.get("男").getSalary() >  map.get("女").getSalary() ? map.get("男").getSalary() : map.get("女").getSalary();
        System.out.println(d);

        list.stream().mapToDouble(People::getSalary).max().ifPresent(System.out::println);
        System.out.println(list.stream().mapToDouble(People::getSalary).max().getAsDouble());

        Map<String, Optional<Integer>> collect = list.stream().collect(Collectors.groupingBy(People::getSex,
                Collectors.mapping(People::getAge, Collectors.maxBy(Comparator.comparing(Function.identity())))));
        collect.get("男").ifPresent(System.out::println);




    }


    /**
     * 男生数量 女生数量
     */
    @Test
    public void example3() {
        list.stream().collect(Collectors.groupingBy(People::getSex,Collectors.counting())).forEach((k,v) -> System.out.println(k + ":" + v));
        System.out.println("男:" + list.stream().filter(people -> people.getSex().equals("男")).count());
        System.out.println("男:" + list.stream().filter(people -> people.getSex().equals("男")).collect(Collectors.counting()));
        System.out.println("男:" + list.stream().filter(people -> people.getSex().equals("男")).collect(Collectors.reducing(0L, e -> 1L, Long::sum)));
        System.out.println("男:" + list.stream().filter(people -> people.getSex().equals("男")).collect(Collectors.toList()).size());
        System.out.println("男:" + list.stream().collect(Collectors.groupingBy(People::getSex)).get("男").size());
        long count= list.stream().collect(Collectors.collectingAndThen(Collectors.groupingBy(People::getSex),map -> map.get("男").stream().count()));
        System.out.println(count);
    }

    /**
     * 平均
     */
    @Test
    public void example4() {
        double avg = list.stream().collect(Collectors.averagingDouble(People::getSalary));
        System.out.println(avg);
        System.out.println(list.stream().collect(Collectors.summarizingDouble(People::getSalary)).getAverage());
        System.out.println(list.stream().map(People::getSalary).reduce((s1,s2) -> s1 + s2).flatMap(sum -> Optional.of(sum / list.size())).get());
    }

    /**
     * 排序处理null
     */
    @Test
    public void example5() {
        list.add(null);
        list.sort(Comparator.nullsFirst(Comparator.comparing(People::getSalary))); // null 数据排前面
        list.sort(Comparator.nullsLast(Comparator.comparing(People::getSalary)));
        CopyOnWriteArrayList list2 = list.stream().sorted(Comparator.nullsLast(Comparator.comparing(People::getSalary))).collect(Collectors.toCollection(CopyOnWriteArrayList::new));
        list2.forEach(System.out::println);
        Collections.sort(list,Comparator.nullsFirst(Comparator.comparing(People::getSalary)));
    }

    /**
     * 分组
     */
    @Test
    public void example6() {
        Map<Boolean, Map<Integer, List<People>>> map = list.stream().collect(
                Collectors.partitioningBy(p -> p.getSex().equals("男"), Collectors.groupingBy(People::getAge)));
        map.get(true).get(30).forEach(System.out::println); // 30岁男生集合
        map.get(false).get(20).forEach(System.out::println); // 20岁女生集合

        Map<String, Map<Integer, List<People>>> map1 = list.stream().collect(
                Collectors.groupingBy(People::getSex, Collectors.groupingBy(People::getAge)));
        map1.get("男").get(30).forEach(System.out::println); // 30岁男生集合

        Map<String,People> map2 =
        list.stream().collect(Collectors.groupingBy(People::getSex,Collectors.collectingAndThen(Collectors.maxBy(Comparator.comparing(People::getSalary)),Optional::get)));
        System.out.println(map2.get("男")); // 按照性别筛选工资最高的男生

        // 按照性别筛选工资最高的男生
        System.out.println(list.stream().sorted(Comparator.comparing(People::getSalary)).collect(Collectors.toMap(People::getSex,Function.identity(),(p1,p2) -> p2)).get("男"));

        Map<String,Set<Integer>> map3 =
        list.stream().collect(Collectors.groupingBy(People::getSex,Collectors.mapping(People::getAge,Collectors.toSet())));
        map3.get("男").forEach(System.out::println); // 按照性别筛选有哪些年龄

        list.stream().filter(people -> people.getSex().equals("男")).map(People::getAge).collect(Collectors.toSet()).forEach(System.out::println); // 按照性别筛选有哪些年龄

        // 按照年龄筛选分别对应有哪些人
        list.stream().collect(Collectors.toMap(People::getAge,People::getName,String::concat)).forEach((k,v) -> System.out.println(k + ":" + v));
        list.stream().collect(Collectors.toMap(People::getAge,People::getName,(name1,name2) -> name1 + "," + name2)).forEach((k,v) -> System.out.println(k + ":" + v));
        Map<Integer,String> map4 = list.stream().collect(Collectors.groupingBy(People::getAge,
                Collectors.reducing("",People::getName,(name1, name2) -> name1.length() == 0 ? name2 : name1 + "," + name2)));
        System.out.println(map4.get(30));

        // 集合名字最长
        System.out.println(list.stream().map(p -> p.getName().length()).reduce(0,Integer::max));

        list.stream().map(People::getName).collect(Collectors.groupingBy(String::length,Collectors.counting())).forEach(
                (k,v) -> System.out.println(k + ":" + v)
        ); // 根据名字长度分组
    }

    /**
     * flatMap(Optional/Stream)
     */
    @Test
    public void example7() {
        list.stream().map(People::getName).flatMap(str -> {
            List<String> stringList = new ArrayList<>();
            for(int i = 0; i < str.length(); i++) {
                String temp = String.valueOf(str.charAt(i));
                stringList.add(temp);
            }
            return stringList.stream();
        }).distinct().collect(Collectors.toList()).forEach(System.out::println);

        double d = list.stream().filter(people -> people.getName().equals("张杰")).mapToDouble(People::getSalary).sum();
        System.out.println(Optional.ofNullable(d).flatMap(e -> Optional.of(e + 5)).flatMap(e -> Optional.of(e - 2)).get()); // 张杰的薪资涨5后又减2
    }
    /**
     * 字符串处理
     */
    @Test
    public void example8() {
       // list.stream().sorted(Comparator.comparing(People::getAge).reversed().thenComparing(People::getSalary));
        System.out.println(list.stream().map(People::getName).collect(Collectors.joining(",","(",")")));
        Collector<People,StringJoiner,String> collector = Collector.of(
                () -> new StringJoiner(","),
                (stringJoiner,people) -> stringJoiner.add(people.getName()),
                StringJoiner::merge,
                StringJoiner::toString
        );
        System.out.println(list.stream().collect(collector));

        StringJoiner field = new StringJoiner(",");
        list.stream().map(People::getName).forEach(field::add);
        System.out.println(field.toString());

        System.out.println(list.stream().map(People::getName).reduce((name1,name2) -> name1.concat(",").concat(name2)).orElse(""));

    }
}
```