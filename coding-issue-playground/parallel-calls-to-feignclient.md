# Parallel calls to feignClient

> background description: if we want to call feignClient, but one request takes too long, so how to we call them in parallel to save more time.

### if we call use default method

```java
// externalService include feignClient that call third-party api
CompletableFuture<Map<String, BigDecimal>> futureA =
        CompletableFuture.supplyAsync(() -> externalService.fetchNumericData(keys));

CompletableFuture<Map<String, String>> futureB =
        CompletableFuture.supplyAsync(() -> externalService.fetchTextData(keys));

CompletableFuture<Map<String, List<GenericItem>>> futureC =
        CompletableFuture.supplyAsync(() -> externalService.fetchGroupedData(keys, DEFAULT_CONFIG));

CompletableFuture
        .allOf(futureA, futureB, futureC)
        .join();

```

this method is not good, from logic level, yes, it can work, but from real production env, it's not a good design.

there are three hidden risks

1.  if we don't pass specific Excutor, then this function will use ForkJoinPool.commonPool(),  externalService.fetchNumericData(keys)) is an HTTP call

    but ForkJoinPool is not suitable for blocking I/O , ForkJoinPool is designed for CPU computing
2. **The number of threads cannot be controlled:** the default number of threads in commonPool ≈ the number of CPU cores, If you request three futures at a time\
   Concurrent 50 requests = 150 blocking tasks\
   commonPool was completely wiped out
3. **The exception & timeout cannot be controlled:** CompletableFuture.allOf().join(), if anyone Future freezes -> all futures will freeze, no timeout, no fallback, it's very difficult to locate when problems occur.

### how about static executor？

okay, if we need a specific Executor, then how about we create a new static executor, then pass it to `supplyAsync()`&#x20;

The answer is still not good, there are four hidden risks.

1. **ThreadPool cannot be closed gracefully：**&#x63;ause if springBoot stops, the threadpool that we created mannually won't be closed automatically. then we may met: the application stucks/JVM forces an exit/the container failed to stop elegantly.
2. **Cannot be uniformly configured and monitored:** thread names are uncontrollable, unable to access Spring Actuator, the numbers of thread cannot be adjusted through the configuration file.
3. **Easily misused (multi-instance problem):** if we use static threadpool and write it in controller and service, each bean instance will create a new thread pool, very dangerous in some scenarios.
4. **Cannot distinguish the buisness threadpool:** like in the future, if we have more threadpool, we create everything by ourseleves, then these threadpool will get out of control.

### Correct version

okay, after we discussion, let's see the correct version to call feignClient in parallel.

{% stepper %}
{% step %}
create configuration class for threadPoolpackage

{% code overflow="wrap" %}
```java
package com.example.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

import java.util.concurrent.Executor;

@Configuration
public class ExecutorConfig {

    @Bean(name = "externalExecutor")
    public Executor externalExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(6);
        executor.setMaxPoolSize(6);
        executor.setQueueCapacity(200);
        executor.setThreadNamePrefix("external-");
        executor.initialize();
        return executor;
    }
}
```
{% endcode %}
{% endstep %}

{% step %}
call in parallel in service

```java
package com.example.service;

import com.example.dto.ExternalPriceDTO;
import com.example.dto.ProductDTO;
import jakarta.annotation.Resource;
import org.springframework.stereotype.Service;

import java.math.BigDecimal;
import java.util.List;
import java.util.Map;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.Executor;
import java.util.concurrent.TimeUnit;

@Service
public class ProductService {

    @Resource(name = "externalExecutor")
    private Executor externalExecutor;

    @Resource
    private ExternalService externalService;

    public void enrichProductDTOs(
            List<ProductDTO> products,
            List<String> productKeys,
            String defaultType
    ) {

        CompletableFuture<Map<String, List<ExternalPriceDTO.Pricing>>> priceFuture =
                CompletableFuture.supplyAsync(
                                () -> externalService.queryPrice(productKeys, defaultType),
                                externalExecutor
                        )
                        .orTimeout(15, TimeUnit.SECONDS)
                        .exceptionally(ex -> Map.of());

        CompletableFuture<Map<String, BigDecimal>> moqFuture =
                CompletableFuture.supplyAsync(
                                () -> externalService.queryMoq(productKeys),
                                externalExecutor
                        )
                        .orTimeout(15, TimeUnit.SECONDS)
                        .exceptionally(ex -> Map.of());

        CompletableFuture<Map<String, String>> stockFuture =
                CompletableFuture.supplyAsync(
                                () -> externalService.queryStock(productKeys),
                                externalExecutor
                        )
                        .orTimeout(15, TimeUnit.SECONDS)
                        .exceptionally(ex -> Map.of());

        CompletableFuture.allOf(priceFuture, moqFuture, stockFuture).join();

        Map<String, List<ExternalPriceDTO.Pricing>> priceMap = priceFuture.join();
        Map<String, BigDecimal> moqMap = moqFuture.join();
        Map<String, String> stockMap = stockFuture.join();

        for (ProductDTO dto : products) {
            String key = dto.getProductKey();

            String priceKey = findKeyByPrefix(priceMap, key);
            String moqKey = findKeyByPrefix(moqMap, key);
            String stockKey = findKeyByPrefix(stockMap, key);

            dto.setPrice(
                    priceKey == null
                            ? "N/A"
                            : priceMap.get(priceKey).stream()
                                      .findFirst()
                                      .map(p -> p.amount() + " " + p.currency())
                                      .orElse("N/A")
            );

            dto.setMoq(moqKey == null ? BigDecimal.ZERO : moqMap.get(moqKey));
            dto.setStock(stockKey == null ? "N/A" : stockMap.get(stockKey));
        }
    }

    private <T> String findKeyByPrefix(Map<String, T> map, String prefix) {
        return map.keySet().stream()
                .filter(k -> k.startsWith(prefix))
                .findFirst()
                .orElse(null);
    }
}

```
{% endstep %}
{% endstepper %}

Summary:

1. manage threadPool by Spring: avoid to `new ThreadPoolExecutor` , shutdown elegantly, define thread's name uniformly
2. multi external api call in parallel: The total time consumption ≈ the time consumption of the slowest interface
3. timeout + fallback defense: like use `orTimeout` ,`exceptionally -> Map.of()`&#x20;
4. in controller level, we don't need @Async/Callable.

### Token issue

when we use the template like above, we will met an token issue.&#x20;

the token actually is stored in SecurityContextHolder from Spring Security.

1.  **SecurityContextHolder is bonded with thread.** \
    By default, Spring sets a SecurityContext in same thread for each request but when we call CompletableFuture. SupplyAsync (...). Or when called by other asynchronous thread pools:

    Theads in the thread pool are not the original requesting threads， So SecurityContextHolder doesn't propagate automatically.

    Then the result is SecurityContextHolder.getContext().getAuthentication(); // may be null\
    So, extractToken() returns null\
    log.error("Unauthorized")\
    Concurrent multi-threading will result in Unauthorized

then we have three solutions

1.  pass SecurityContext mannually

    ```java
    SecurityContext context = SecurityContextHolder.getContext();

    CompletableFuture.supplyAsync(() -> {
        try {
            SecurityContextHolder.setContext(context);
            return externalService.getProductInfo(productKeys);
        } finally {
            SecurityContextHolder.clearContext(); //Avoid contamination of the thread pool's upper and lower text

        }
    }, externalExecutor);

    ```
2.  `DelegatingSecurityContextExecutor`（Offical suggestion）

    ```java
    Executor delegatingExecutor =
            new DelegatingSecurityContextExecutor(externalExecutor);

    CompletableFuture.supplyAsync(
            () -> externalService.getProductMoq(productKeys),
            delegatingExecutor
    );

    ✅ Spring Security official solution

    ✅ set/clear automatically

    ✅ code is extremely clean

    ✅ will not contaminate the thread pool

    ✅ supports concurrency
    ```
3.  get token from controller, then pass it to service

    ```java
    //in controller
    String bearerToken = TokenUtils.getBearerToken();
    productService.enrichProductDTOs(products, bearerToken, ...);

    //in service
    externalService.getProductMoq(productKeys, bearerToken);


    //this solution isn't good, the token needs to be leaked

    ```

### FeignClient call in parallel

By default, FeignClient calls are synchronously blocked, each call is blocked and waits for a response

1. Use CompletableFuture +Spring ThreadPool

```java
//use the Executor we define before, externalExecutor
List<String> itemIdList = List.of(itemIds.split(","));
List<CompletableFuture<Void>> futures = new ArrayList<>();

for (String itemId : itemIdList) {
    CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
        JsonNode info = externalClient.queryItemInfo(region, customerId, itemId, "", 1, 10);

        if (info != null && !info.get("data").isNull()) {
            JsonNode record = info.path("data").path("records").path(0);
            if (!record.isMissingNode()) {
                JsonNode qtyNode = record.path("quantity");
                JsonNode codeNode = record.path("code");

                if (!qtyNode.isMissingNode() && !codeNode.isMissingNode()) {
                    String code = codeNode.asText();
                    BigDecimal quantity = qtyNode.decimalValue();

                    // 并发安全更新共享 Map
                    synchronized (itemQuantityMap) {
                        itemQuantityMap.put(code, quantity);
                    }
                }
            }
        }
    }, externalExecutor); // the threadpool which managed by Spring
    futures.add(future);
}

// wait for all task finished
CompletableFuture.allOf(futures.toArray(new CompletableFuture[0])).join();

```

2. Use Async Feign

```java
//Feign Client level
@FeignClient("external-service")
public interface ExternalClient {

    @RequestLine("GET /itemInfo")
    CompletableFuture<JsonNode> getItemInfoAsync(
            @Param("region") String region,
            @Param("customerId") String customerId,
            @Param("itemId") String itemId,
            @Param("page") int page,
            @Param("size") int size
    );
}

//in service
List<String> itemIdList = List.of(itemIds.split(","));
List<CompletableFuture<Void>> futures = new ArrayList<>();

for (String itemId : itemIdList) {
    CompletableFuture<Void> future = externalClient.getItemInfoAsync(region, customerId, itemId, 1, 10)
            .thenAccept(info -> {
                if (info != null && !info.get("data").isNull()) {
                    JsonNode record = info.path("data").path("records").path(0);
                    if (!record.isMissingNode()) {
                        JsonNode qtyNode = record.path("quantity");
                        JsonNode codeNode = record.path("code");
                        if (!qtyNode.isMissingNode() && !codeNode.isMissingNode()) {
                            String code = codeNode.asText();
                            BigDecimal quantity = qtyNode.decimalValue();

                            // 并发安全更新共享 Map
                            synchronized (itemQuantityMap) {
                                itemQuantityMap.put(code, quantity);
                            }
                        }
                    }
                }
            });
    futures.add(future);
}

// wait all async task finished
CompletableFuture.allOf(futures.toArray(new CompletableFuture[0])).join();


```
