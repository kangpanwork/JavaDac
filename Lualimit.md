Lua 是一种轻量小巧的脚本语言，定义注解
```Java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Limit {

    // 方法名为key
    String key() default "";

    // 时间，单位秒
    int period();

    // 限制访问次数
    int count();
}
```
限流切面
``` Java
@Aspect
@Component
@Slf4j
public class LimitAspect {
    private final RedisTemplate<Object, Object> redisTemplate;

    public LimitAspect(RedisTemplate<Object, Object> redisTemplate) {
        //设置String key 和value序列化模式
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        this.redisTemplate = redisTemplate;
    }

    @Pointcut("@annotation(com.kangpan.annotation.Limit)")
    public void pointcut() {
    }

    @Around("pointcut()")
    public Object around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        MethodSignature methodSignature = (MethodSignature) proceedingJoinPoint.getSignature();
        Method method = methodSignature.getMethod();
        Limit limit = method.getAnnotation(Limit.class);
        String key = limit.key();
        ImmutableList<Object> keys = ImmutableList.of(key);
        String lua = "local num"
                + "\n local key_local = redis.call('setnx',KEYS[1],0)" // KEY存在不做任何事情 不存在赋值为0
                + "\n if (tonumber(key_local) == 1)" // ==0 表示KEY存在 没有赋值  ==1 表示KEY不存在 赋值0了
                + "\n then"
                    + "\n redis.call('incr',KEYS[1])" // 存储在key中的数字值加1 如果key不存在 key的值先被设置为0 然后再执行加1操作
                    + "\n num = redis.call('get',KEYS[1])" // 得到Key的值
                        + "\n if (tonumber(num) == 1)" // 第一次赋值
                        + "\n then"
                            + "\n redis.call('expire',KEYS[1],ARGV[2])" // 设置有效时间
                        + "\n end"
                    + "\n return tonumber(num);"
                + "\n else"
                    + "\n redis.call('incr',KEYS[1])"
                    + "\n num = redis.call('get',KEYS[1])"
                    + "\n return tonumber(num);"
                + "\n end";

        RedisScript<Number> redisScript = new DefaultRedisScript<>(lua, Number.class);
        Number number = redisTemplate.execute(redisScript, keys, limit.count(), limit.period());
        if (null != number && number.intValue() <= limit.count()) {
            log.info("方法{}: 第{}次访问 有效时间为 {}", limit.key(), number, redisTemplate.getExpire(limit.key()));
            return proceedingJoinPoint.proceed();
        } else {
            throw new BadRequestException("访问次数受限制");
        }
    }
}
```
`BadRequestException` 类
```Java
public class BadRequestException extends RuntimeException {
    public BadRequestException(String message) {
        super( message );
    }

    public BadRequestException() {
        super();
    }

    public BadRequestException(String message, Throwable cause) {
        super( message, cause );
    }

    public BadRequestException(Throwable cause) {
        super( cause );
    }
}

```
捕获 `BadRequestException` 处理类
``` Java
@Slf4j
@RestControllerAdvice(basePackages = "com.kangpan.controller")
@Order(Ordered.HIGHEST_PRECEDENCE)
public class GlobalExceptionHandler {

    @ExceptionHandler({ValidationException.class, BadRequestException.class})
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ResponseEntity handleException(Exception e) {
        return ResponseEntity.builder().time(LocalDateTime.now()).message(e.getMessage()).status(HttpStatus.BAD_REQUEST.value()).build();
    }
}
```
自定义 `ResponseEntity` 
```Java
@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class ResponseEntity<T> {
    /**
     * 返回信息
     */
    private String message;

    /**
     * 返回码
     */
    private Integer status;

    /**
     * 返回数据
     */
    private T data;

    /**
     * 时间
     */
    @JsonFormat(pattern="yyyy:MM:dd HH:mm:ss",timezone = "GMT+8")
    private LocalDateTime time;
}
```
应用 `controller` 
``` Java
    @PostMapping("/limitBuy")
    @Limit(key = "limitBuy", count = 10, period = 60)
    @ApiOperation("限次数购买")
    @ResponseBody
    public ResponseEntity limitBuy(Order order) {
        orderService.save(order);
        return ResponseEntity.builder().message("buy success").build();
    }
```