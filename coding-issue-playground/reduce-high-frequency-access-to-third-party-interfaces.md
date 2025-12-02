---
description: >-
  when you have a third-party interfaces and need to get data in time, however
  the third-party doesn't like you access frequently, how do we should do.
---

# Reduce high frequency access to third-party interfaces

Firstly, we cannot store that in the table, cause access frequently to DB is also not a great solution, and for redis or some cache we also cannot use, because we need the data in time.

### Pull to Push

that means let third-party inform you when their data is changed.

{% stepper %}
{% step %}
#### if third-party api support **Webhook / CallBack**

then we just need to expose a callback interface

```
@RestController
@RequestMapping("/callback")
public class ThirdPartyCallbackController {

    @PostMapping("/update")
    public void onUpdate(@RequestBody ThirdPartyData data) {
        // store into cache or update local cache
    }
}
```
{% endstep %}

{% step %}
#### if third-party api support WebSocket / SSE

then we just need to build connection

```
WebClient client = WebClient.create("wss://...");
client.get()
      .retrieve()
      .bodyToFlux(ThirdPartyData.class)
      .subscribe(cache::update);
```
{% endstep %}

{% step %}
#### if third-party api provided ETag / If-None-Match / If-Modified-Since

then send request with Etag, and you will got 304 Not Modified when server doesn't have updated data.

```
WebClient client = WebClient.create();
client.get()
      .uri("https://api/")
      .header("If-None-Match", lastEtag)
      .retrieve()
      .toEntity(String.class)
```
{% endstep %}
{% endstepper %}

### Optimize polling to reduce "nonsense requests" globally

that means we call api with the real request

#### Dynamic Polling

Data change frequently -> quick polling

Data does not change for a long time → automatically reduces the frequency

more details like this

```
after the first change：poll each 5s
no change for 5 consecutive times：reduce to poll each 30s
no change for 5 consecutive times：reduce to poll each 1min
checked the change：reset to poll each 5s
```

#### Multi-Level Cache

well, if you still think that call so much times, then we can use cache, but not only one cache.

1. local memory cache(Caffeine): Millisecond access
2. Redis Distributed cache: Different instances share data
3. Third party interface: only requested when cache expires/needs to be flushed

```
// Caffeine configuration for SpringBoot.
@Bean
public Caffeine<Object, Object> caffeineConfig() {
    return Caffeine.newBuilder()
            .expireAfterWrite(Duration.ofSeconds(5));
}

```

#### Change Detecting Polling

well, if a good third-party provided some version info, like version\_number / last\_update\_time / revision

then we can use that

First, request lightweight interface for version number first → Request big data when version number changes.

But it's hard because we're asking for someone else's, not ours.

