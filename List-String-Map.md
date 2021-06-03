日常工作中经常和 集合、数组、Map、字符串互相转换 打交道，这里记录一些  [Guava](https://github.com/google/guava/wiki) 的用法。


使用 `StringBuilder` 
```java
    /**
     * listToStringByStringBuilder
     *
     * @param list 分隔的集合
     * @param delimiter 分隔符
     * @return String 结果
     */
    public static String listToStringByStringBuilder(List<String> list, String delimiter) {
        StringBuilder stringBuilder = new StringBuilder();
        list.forEach(element -> stringBuilder.append(element).append(delimiter));
        stringBuilder.delete(stringBuilder.length()- delimiter.length(),stringBuilder.length()); // delete(start, end);
        return  stringBuilder.toString();
    }
```
使用 `Collectors.joining`
```java
    /**
     * listToStringByCollectorsJoining
     *
     * @param list 分隔的集合
     * @param delimiter 分隔符号
     * @param prefix 前缀符号
     * @param suffix 后缀符号
     * @return String 结果
     */
    public static String listToStringByCollectorsJoining(List<String> list, String delimiter, String prefix, String suffix) {
        return list.stream().collect(Collectors.joining(delimiter, prefix, suffix ));
    }
```
使用 `StringJoiner`
```java
   /**
     * listToStringByStringJoiner
     *
     * @param list 分隔的集合
     * @param separator 分隔符
     * @param prefix 前缀符号
     * @param suffix 后缀符号
     * @return String 结果
     */
    public static String listToStringByStringJoiner(List<String> list, String separator, String prefix, String suffix) {
        StringJoiner stringJoiner = new StringJoiner(separator,prefix,suffix);
        list.forEach(element -> stringJoiner.add(element));
        return  stringJoiner.toString();
    }
```
使用 `Joiner` ， `String.join()` 方法 也可以对集合或者数组进行分隔
```java
    /**
     * arrayToString
     *
     * @param parts 分隔的数组
     * @param separator 分隔符
     * @param skipNull 是否跳过NULL
     * @param useForNull 是否替换NULL
     * @param nullText 替换NULL的值
     * @return String 结果
     */
    public static String arrayToString(Object[] parts,String separator,boolean skipNull,boolean useForNull,String nullText) {
        if (skipNull) {
            return Joiner.on(separator).skipNulls().join(parts);
        }
        if (useForNull) {
            return Joiner.on(separator).useForNull(nullText).join(parts);
        }
        return Joiner.on(separator).join(parts); //   String.join() 也可以
    }
```
`keyValueSeparator` 为 `map` 中 `key` 和 `value` 的分隔符
```java
    /**
     * mapToString
     *
     * @param map 分隔的map
     * @param separator 分隔符
     * @param keyValueSeparator 键值对分隔符
     * @return String 结果
     */
    public static String mapToString(Map map, String separator, String keyValueSeparator) {
        return Joiner.on(separator).withKeyValueSeparator(keyValueSeparator).join(map);
    }
```
字符串 `aabbcc` ，按照长度为2，会分隔元素为 `[aa] [bb] [cc]` 的集合 
```java
    /**
     * stringToFixedLengthList
     *
     * @param sequence 分隔的字符串 String 实现 CharSequence 接口
     * @param length 分隔的长度
     * @return List<String> 结果
     */
    public static List<String> stringToFixedLengthList(CharSequence sequence, int length) {
        Iterable<String> iterable =  Splitter.fixedLength(length).split(sequence);
        List<String> list = new ArrayList<>();
        for (Iterator<String> iterator = iterable.iterator(); iterator.hasNext();) {
            list.add(iterator.next());
        }
        return list;
    }
```
字符串转集合，方法有很多种。这里举两个，说明： `trimResults()` 去掉元素左右空字符串，`omitEmptyStrings` 去掉分隔符号之间的空元素。举个例子：
`" k ;;p"` ，`trimResults`之后得到 `[k] [空] [p]` 三个元素，加上 `omitEmptyStrings ` 会去除空元素。
```java
    /**
     * stringSplitToList
     *
     * @param separator 分隔符
     * @param sequence 分隔的字符串
     * @return List<String> 结果
     */
    public static List<String> stringSplitToList(String separator,CharSequence sequence) {
        Iterable<String> iterable = Splitter.on(separator).trimResults().omitEmptyStrings().split(sequence);
        List<String> list = new ArrayList<>();
        Iterator<String> iterator = iterable.iterator();
        while (iterator.hasNext()) {
            list.add(iterator.next());
        }
        return Splitter.on(separator).trimResults().omitEmptyStrings().splitToList(sequence);
        // return list;
    }
```
使用 `java8` 新特性
```java
    /**
     * StringToList
     *
     * @param separator 分隔符
     * @param str 分隔的字符串
     * @return List 结果
     */
    public static List StringToList(String separator, String str) {
        return Arrays.stream(str.split(separator)).filter(e -> !e.isEmpty()).collect(Collectors.toList());
        // return Splitter.on(separator).trimResults().splitToList(str);
    }
```
字符串 `a,b,c,d,e,f,g` 限制长度为3，得到集合元素为 ` [a] [b] [c,d,e,f,g] `
```java
    /**
     * limitSplitToList
     *
     * @param separator 分割符
     * @param length 限制长度
     * @param sequence 分隔的字符串
     * @return List<String> 结果
     */
    public static List<String> limitSplitToList(String separator, int length, CharSequence sequence) {
        return Splitter.on(separator).limit(length).splitToList(sequence);
    }
```
字符串 `a,b:c,d;e` 有不同分隔符， 逗号、 冒号、 分号等，利用正则表达式 `[,|:|;]` 分隔
```java
    /**
     * multiLimitSplitToList
     *
     * @param separatorPattern 多个分隔符
     * @param sequence 分隔的字符串
     * @return List<String>
     */
    public static List<String> multiLimitSplitToList(String separatorPattern, CharSequence sequence) {
        return Splitter.onPattern(separatorPattern).splitToList(sequence);
    }
```
字符串 `name=kangPan;age=10;six=boy` 最后不能带分隔符，否则报错。分隔符是分号，键值对分隔符是等号，会拆分
`key是name,value是kangPan`等键值对的`map`
```java
    /**
     * StringToMap
     *
     * @param separator 分隔符
     * @param withKeyValueSeparator 键值对分割符
     * @param sequence 分隔的字符串
     * @return Map 结果
     */
    public static Map StringToMap(String separator,String withKeyValueSeparator, CharSequence sequence) {
        return Splitter.on(separator).withKeyValueSeparator(withKeyValueSeparator).split(sequence);
    }

```
