# DelegatingSecurityContextExecutor

if we run some async task with ThreadPool, how do we pass the token which is stored in Spring Security.

**Conclusion First: DelegatingSecurityContextExecutor**

let's get it step by step

1. we pass the token to controller, then will start a HTTP request, it's our main thread.
2. then the token will be stored in SecurityContextHolder
3. then we start a new threadpool to run sub tasks.&#x20;

ok, in here, if we don't handle, the token will lose. bcs&#x20;

> `SecurityContextHolder.getContext()`

this is a ThreadLocal, and bind with current Thread which is our main thread.

so the new ThreadLocal of new thread is empty.



### What does DelegatingSecurityContextExecutor use for?

**one-sentence answer: Pass the SecurityContext of the calling thread to the execution thread**

In other word:

1. main Thread has a SecurityContext
2. we wrap this SecurityContext into Runnable/Callable
3. when sub Thread execute, it put this context to their ThreadLocal
4. and will clean it after sub Thread finish.

one important point for this one is : DelegatingSecurityContextExecutor doesn't share SecurityContext to other thread, it copy one context to each thread.

> It still is possible to lose the token when we use DelegatingSecurityContextExecutor, like these two scenario:
>
> 1. the SecurityContext doesn't exist when DelegatingSecurityContextExecutor try to pass it
> 2. after you get the SecurityContext, you may try to change it.

### Simple case to use DelegatingSecurityContextExecutor&#x20;

how do we use it in code?

{% stepper %}
{% step %}
### Spring Security store the token

like if you send a request with the token, what is this token passed?

```
GET /api/products/123
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6...
```

then in the filter chain of Spring Security, like BearerTokenAuthenticationFilter will do

```
Authentication authentication = authenticate(token);
SecurityContext context = SecurityContextHolder.createEmptyContext();
context.setAuthentication(authentication);
SecurityContextHolder.setContext(context);
```

yeah, we can see now the token is stored in context
{% endstep %}

{% step %}
### we can see the token in Controller level&#x20;

```
@RestController
@RequiredArgsConstructor
public class ProductController {
    private final ProductService productService;

    @GetMapping("/api/products/moq")
    public Map<String, BigDecimal> getMoq(@RequestParam String productIds) {
        // here we can get the token explicitly
        String token = SecurityContextHolder.getContext().getAuthentication();
        return productService.getProductInfo(productIds);
    }
}
```

now we are in main Thread
{% endstep %}

{% step %}
### Create DelegatingSecurityContextExecutor in Service Level

```
@Service
@RequiredArgsConstructor
public class ProductService {
    // a threadPool, we can define with Bean
    private final ThreadPoolTaskExecutor thirdPartyExecutor;

    public Map<String, BigDecimal> getProductMoq(String productIds) {

        // ⚠️ key: SecurityContext already exist
        Executor securityExecutor =
                new DelegatingSecurityContextExecutor(thirdPartyExecutor);

        List<String> productIdList = List.of(productIds.split(","));
        Map<String, BigDecimal> result = new ConcurrentHashMap<>();

        List<CompletableFuture<Void>> futures = productIdList.stream()
                .map(productId ->
                        CompletableFuture.runAsync(
                                () -> callThirdPartyAndPutResult(productId, result),
                                securityExecutor
                        )
                )
                .toList();

        CompletableFuture
                .allOf(futures.toArray(new CompletableFuture[0]))
                .join();

        return result;
    }

```

in `Executor securityExecutor = new DelegatingSecurityContextExecutor(thirdPartyExecutor);`  will execute `SecurityContextHolder.getContext()` right now, and catch the info&#x20;

and why it doesn't affect next request, bcs after current sub task finished its task, it will be cleaned&#x20;

```
finally {
    SecurityContextHolder.setContext(originalSecurityContext);
}
```

this is very important key to keep thread security.
{% endstep %}
{% endstepper %}

