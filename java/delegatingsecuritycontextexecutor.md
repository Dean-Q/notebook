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

one-sentence answer: Pass the SecurityContext of the calling thread to the execution thread

