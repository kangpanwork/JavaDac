`org.springframework.util` 包下的 `MultiValueMap` 接口

``` Java
void add(K key, @Nullable V value);
```

看下 `LinkedMultiValueMap` 实现

``` Java
@Override
public void add(K key, @Nullable V value) {
	List<V> values = this.targetMap.computeIfAbsent(key, k -> new LinkedList<>());
	values.add(value);
}
```

`targetMap` 在 `LinkedMultiValueMap` 构造方法的时候初始化了

``` Java
private final Map<K, List<V>> targetMap;
public LinkedMultiValueMap() {
    this.targetMap = new LinkedHashMap<>();
}
```
介绍下 `computeIfAbsent`  当`get(key)` 是 `null` 的时候 我们 `put` 新的 `value`  如果不是 那么直接返回 `get(key)` 也就是 `LinkedList`
``` Java
   default V computeIfAbsent(K key,
            Function<? super K, ? extends V> mappingFunction) {
        Objects.requireNonNull(mappingFunction);
        V v;
        if ((v = get(key)) == null) {
            V newValue;
            if ((newValue = mappingFunction.apply(key)) != null) {
                put(key, newValue);
                return newValue;
            }
        }

        return v;
    }
```
举个例子
``` Java
    MultiValueMap<String,String> map = new LinkedMultiValueMap<>();
    map.add("k","1");
    map.add("k","2");
```
`map` 对应的 `key` 是 `k`，  对应的 `value` 是一个 `LinkedList` ，里面元素有 `1` 和 `2`

`MultiValueMap` 应用在 `SpringFactoriesLoader` 这个类检索 `META-INF/spring.factories` 文件

```Java
	private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
		MultiValueMap<String, String> result = cache.get(classLoader);
		if (result != null) {
			return result;
		}

		try {
			Enumeration<URL> urls = (classLoader != null ?
					classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
					ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
			result = new LinkedMultiValueMap<>();
			while (urls.hasMoreElements()) {
				URL url = urls.nextElement();
				UrlResource resource = new UrlResource(url);
				Properties properties = PropertiesLoaderUtils.loadProperties(resource);
				for (Map.Entry<?, ?> entry : properties.entrySet()) {
					String factoryTypeName = ((String) entry.getKey()).trim();
					for (String factoryImplementationName : StringUtils.commaDelimitedListToStringArray((String) entry.getValue())) {
						result.add(factoryTypeName, factoryImplementationName.trim());
					}
				}
			}
			cache.put(classLoader, result);
			return result;
		}
		catch (IOException ex) {
			throw new IllegalArgumentException("Unable to load factories from location [" +
					FACTORIES_RESOURCE_LOCATION + "]", ex);
		}
	}
```