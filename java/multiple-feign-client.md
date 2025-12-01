# Multiple Feign Client

how to implement multiple Feign Clients in one repo

if you use @Configuration in a Feign Client Configuration class, this class will be considered as a gobal configuration, so if we have multiple feign client, and each feign client has their own configuration, only this gobal configuration will work, then how do we handle this.

```
feign/
 ├── client/
 │    ├── ClientA.java            # clients
 │    └── ClientB.java
 ├── config/
 │    ├── ClientAConfig.java            # configurations for Speicific Client
 │    └── ClientBConfig.java  
 │    └── CommonFeignConfig.java

```

the most important thing is don't use @Configuration annoation.

### CommonFeignConfig

you can configure some common things in this class, like log, proxy and so on.

```
public class CommonFeignConfig {

    @Bean
    Logger.Level loggerLevel() {
        return Logger.Level.BASIC;
    }

    @Bean
    Request.Options feignTimeouts() {
        return new Request.Options(2000, 5000);
    }
}
```

### ClientAConfig

how do we also use common configuration, use @Import ,the token of ClientA comes from `SecurityContextHolder.getContext().getAuthentication()`

```
@Slf4j
@Import(CommonFeignConfig.class)
public class ClientAConfig {
     @Value("${clientA.url}")
      private String clientAUrl;
      
    //attention: if each client has their own Interceptor, need to define different 
    //      interceptor name to dinstinguish.
    @Bean(name = "clientAAuthInterceptor")
    public RequestInterceptor clientAAuthInterceptor() {
        return template -> {
            String baseUrl = clientAUrl;
            String fullUrl = baseUrl + template.url();
            var token = TokenUtils.getBearerToken();

            template.header("Authorization", "Bearer " + token);

            StringBuilder headers = new StringBuilder();
            template.headers()
                    .forEach((k, v) -> headers.append(k)
                            .append("=")
                            .append(v)
                            .append("; "));
            String body = template.body() != null ? new String(template.body(), template.requestCharset()) : "No Body";

            log.info("[ClientA Feign call] {} {} | Headers: {} | Body: {}",
                    template.method(), fullUrl, headers, body);
        };
    }
}

```

### ClientBConfig

the token of ClientB comes from application.yaml configuration

```
@Slf4j
@Import(CommonFeignClientConfig.class)
public class ClientBFeignClientConfig {
    private final ClientBProperties properties;

    @Bean(name = "ClientBAuthInterceptor")
    public RequestInterceptor ClientBAuthInterceptor() {
        return template -> {
            String baseUrl = properties.url();
            String fullUrl = baseUrl + template.url();

            template.header("API-KEY", properties.apiKey());

            StringBuilder headers = new StringBuilder();
            template.headers()
                    .forEach((k, v) -> headers.append(k)
                            .append("=")
                            .append(v)
                            .append("; "));
            String requestBody = template.body() != null ? new String(template.body(), template.requestCharset()) : "No Body";

            log.info("[ClientB Feign call] {} {} | Headers: {} | Body: {}",
                    template.method(), fullUrl, headers, requestBody);
        };
    }

//we have decoder and errordecoder to handle correct response and incorrect response
    @Bean(name = "ClientBFeignDecoder")
    public Decoder ClientBFeignDecoder() {
        return (response, type) -> {
            ObjectMapper mapper = new ObjectMapper();
            JsonNode node = mapper.readTree(response.body()
                    .asInputStream());

            if (node.path("code")
                    .asInt() != 2000) {
                throw new ClientBApiException(
                        node.path("code")
                                .asInt(response.status()),
                        "Failed to call ClientB API via Feign (status: " + node.path("code")
                                .asInt(response.status()) + ", error: " + node.path("msg")
                                .asText("Unknown error")
                );
            }

            ObjectNode result = mapper.createObjectNode();
            result.set("data", node.path("data"));

            log.debug("[ClientB Feign call] response: {} {}", response.status(), result);
            return result;
        };
    }

    @Bean(name = "ClientBErrorDecoder")
    public ErrorDecoder ClientBErrorDecoder() {
        return (methodKey, response) -> {
            try {
                if (response.body() != null) {
                    ObjectMapper mapper = new ObjectMapper();
                    JsonNode node = mapper.readTree(response.body()
                            .asInputStream());
                    return new ClientBApiException(
                            node.path("code")
                                    .asInt(response.status()),
                            "Failed to call ClientB API via Feign (status: " + node.path("code")
                                    .asInt(response.status()) + ", error: " + node.path("msg")
                                    .asText("Unknown error")
                    );
                } else {
                    return new ClientBApiException(
                            response.status(),
                            "Failed to call ClientB API via Feign (status: " + response.status() + "): empty body"
                    );
                }
            } catch (Exception e) {
                return new ClientBApiException(
                        response.status(),
                        "Failed to call ClientB API via Feign (status: " + response.status() + "): " + e.getMessage()
                );
            }
        };
    }
}
```

### ClientA

how do we match ClientAConfiguration to ClientA, use the `configuration` attributes of @FeignClient

```
@FeignClient(
        name = "ClientAClient",
        url = "${ClientA.url}",
        configuration = ClientAConfig.class
)
public interface ClientAClient {
@GetMapping("${ClientA.paths.user}/{userId}")
    JsonNode getUser(@PathVariable("userId") String userId);
}

```

### ClientB

same way as ClientA.
