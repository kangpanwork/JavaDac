`pom `文件引入
```
<!-- https://mvnrepository.com/artifact/com.github.houbb/junitperf -->
<dependency>
    <groupId>com.github.houbb</groupId>
    <artifactId>junitperf</artifactId>
    <version>2.0.7</version>
</dependency>
```
测试 `stringBuilder `
```Java
public class Test {
    @JunitPerfConfig( warmUp = 1000, reporter = {HtmlReporter.class, ConsoleReporter.class})
    public void test() {
        StringBuilder stringBuilder = new StringBuilder();
        for (int i = 0; i < 10000; i++) {
            stringBuilder.append(i);
        }
    }
}
```
看下  `JunitPerfConfig ` 类
``` Java
public @interface JunitPerfConfig {
    int threads() default 1; // 执行时使用多少线程执行
    long warmUp() default 0L; // 准备时间 （单位：毫秒）

    long duration() default 60000L; // 执行时间 (单位：毫秒)  (不包括准备时间) 默认值1分钟

    Class<? extends StatisticsCalculator> statistics() default DefaultStatisticsCalculator.class; // 统计信息

    Class<? extends Reporter>[] reporter() default {ConsoleReporter.class}; // 报告信息
}
```

![](/27.jpg)

参考 [junitperf](https://github.com/houbb/junitperf)
