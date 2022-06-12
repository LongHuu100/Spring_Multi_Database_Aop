Mục đích của AOP (Lập trình hướng cạnh) để thay đổi logic của ứng dụng mà không cần thay đổi code của modul có sẵn thông qua việc chạy các middelware chèn vào trước và sau logic.

Trong ví dụ này là chạy code chuyển giữa nhiều database thông qua AOP Pointcut.

Tạo PointCut cho annotation là com.dynamicdatasource.demo.config.SwitchDataSource.
* Ở đâu dùng annotation (@SwitchDataSource) này thì sẽ được áp dụng point cut như sau.
* 1. Chạy hàm before(JoinPoint joinPoint) để chuyển sang database khác.
* 2. Chạy logic tại nơi khai báo annotation.
* 3. Chạy void after(JoinPoint point) để chuyển về database mặc định.

```
@Pointcut("@annotation(com.dynamicdatasource.demo.config.SwitchDataSource)")
public void annotationPointCut() {
    //default function
}

@Before("annotationPointCut()")
public void before(JoinPoint joinPoint){
    MethodSignature sign =  (MethodSignature) joinPoint.getSignature();
    Method method = sign.getMethod();
    SwitchDataSource annotation = method.getAnnotation(SwitchDataSource.class);
    if(annotation != null){
        AbstractRoutingDataSourceImpl.setDatabaseName(annotation.value());
        logger.info("Switch DataSource to [{}] in Method [{}]", annotation.value(), joinPoint.getSignature());
    }
}

@After("annotationPointCut()")
public void after(JoinPoint point){
    String dbName = AbstractRoutingDataSourceImpl.getDatabaseName();
    if(null != dbName) {
        logger.info("Restore DataSource to [{}] in Method [{}]", dbName, point.getSignature());
        AbstractRoutingDataSourceImpl.removeDatabaseName();
    }
}
```

##### Nếu muốn thay đổi kết quả trả về của một method thì sử dụng @Around
```
@Aspect
@Component
public class MemoizeAspect {
    private WeakHashMap<Object[], Order> cache = new WeakHashMap<Object[], Order>();

    @Around("@annotation(com.aspects.Memoize)")
    public Object handledMemoize(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("Handling memoize");
        Object[] args = pjp.getArgs();
        if (args != null) {
            Object response = cache.get(Arrays.asList(args));
            if (response != null){
                return response;
            }
        }
        Object obj = pjp.proceed();
        if (obj instanceof Order && obj != null) {
            cache.put(Arrays.asList(args), obj);
            obj.setCode("FixCode_JoinPoint");
        }
        return obj;
    }
}
```
