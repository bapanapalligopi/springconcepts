Ability or capacity to recover quickly from difficulties
when you building any kind of systems you can not avoid the failures , we should handle them gracefully

Techniques to handle gracefully
1. Retries-
   -> make retry request certain number of times to the services
   -> each request may or may not get the success
   -> if microservoces up and running we will get the sucess response form service
2. Rate Limiting
   -> controlls the number of requests that services would handle
   -> protecting the against the exssessive traffix
3. Bulkheads
   -> if one services experince high load it can't it can not impcat to the other servie
4. Circuit Breakers
5. fallbacks
   -> alternative actions when some service
   -> like cache
   -> like secondary db
6. Timeouts
    -> wait some time
   -> specific time period
   -> alternative path can be anything
7. Graceful Degradation
   ->
to implement all these we can use Resilience4j
it is simple library ans lightweig
easy integration with springboot
build for functional programming paradidms

Resilience4J Modules
1. Retry Module
2. Rate Limiter Module
3. BulkHead
4. CircuitBreaker Module
5. 
