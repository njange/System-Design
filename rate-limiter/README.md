
# Rate Limiter

A concise, production-oriented Java library demonstrating two common rate-limiting algorithms implemented on top of Redis: a Fixed Window limiter and a Token Bucket limiter. This repository is intended as a reference implementation and a starting point for deploying reliable throttling in distributed services.

## Project Summary

This project implements two server-side rate limiting strategies that are common in API gateways, service meshes, and microservices:

- **Fixed Window**: Counts requests in fixed time windows (e.g., per minute). Simple and efficient but can create bursts at window boundaries.
- **Token Bucket**: Maintains a bucket of tokens where tokens are added at a steady rate and removed on each request. Provides smoother rate enforcement and allows short bursts up to bucket capacity.

Both implementations use Redis as the backing store for counters and token state so they can be used in horizontally scaled systems.

## Key Components

- `FixedWindowRateLimiter.java` — Implementation of the fixed-window algorithm using Redis counters and TTLs to represent windows.
- `TokenBucketRateLimiter.java` — Token-bucket implementation that stores available tokens and refill timestamps in Redis.

## Project Structure

- `src/main/java/io/redis/` — Source code for the rate limiters.
- `src/test/java/io/redis/` — Unit tests validating limiter behavior.
- `pom.xml` — Build configuration (Maven).

## Building and Running Tests

Requires Java 11+ and Maven.

Build and run tests from the repository root:

```bash
mvn clean test
```

The tests exercise expected behaviors (allow, throttle, refill) for both limiter implementations.

## Usage

1. Configure a Redis client (Jedis, Lettuce, or other) and provide it to the limiter constructors.
2. Create a limiter instance tuned to your desired capacity and window/refill policy.
3. Call the limiter check method before processing requests; reject or queue requests when limits are exceeded.

Example (pseudo-code):

```java
// Example: Token bucket allowing 100 requests/second with burst capacity of 200
RedisClient redis = ...; // configure client
TokenBucketRateLimiter limiter = new TokenBucketRateLimiter(redis, "api:tokens", 100, 200);

if (limiter.tryConsume(1)) {
	// process request
} else {
	// reject or respond with 429 Too Many Requests
}
```

## Configuration Options

- Fixed-window: window size (seconds), max requests per window, Redis key namespace.
- Token-bucket: refill rate (tokens/second), bucket capacity, Redis key namespace.

Tune these values based on traffic patterns and acceptable burstiness for your service.

## Testing & Validation

- Unit tests are provided in `src/test/java/io/redis/` to validate algorithm correctness for typical and boundary conditions.
- Run `mvn -Dtest=... test` to run specific tests.

## Contributing

Contributions are welcome. Please open an issue for feature requests or bugs and follow standard GitHub PR workflows. Include tests for any behavior changes.

## Future Improvements

Planned or suggested enhancements to move this project toward production-readiness:

- **Atomicity with Lua scripts**: Use Redis Lua scripts to make multi-key updates atomic and avoid race conditions under high concurrency.
- **Sliding window / leaky bucket**: Add sliding-window or leaky-bucket algorithms for more accurate rate smoothing across window boundaries.
- **Cluster-aware storage & sharding**: Add support for Redis Cluster and strategies for sharded counters/tokens.
- **High-resolution timing**: Replace second-resolution windows with millisecond precision where needed.
- **Non-blocking / async APIs**: Provide async/reactive client integrations (Lettuce reactive, CompletableFuture) for non-blocking servers.
- **Metrics & observability**: Emit Prometheus metrics (allowed/blocked counts, refill rates, latency) and tracing hooks for integration with APM.
- **Dynamic configuration**: Support runtime policy changes (per-tenant or per-endpoint) via config service or Redis-based feature flags.
- **Performance benchmarking**: Add JMH or Gatling benchmarks to quantify performance and latency under load.
- **Fail-open / fail-closed policies**: Provide configurable behavior for Redis outages (choose to fail-open or fail-closed) and circuit-breaker integration.
- **Language/platform adapters**: Provide example adapters or wrappers for common web frameworks (Spring Boot filter, servlet filter, Netty handler).

## License

This project is provided under an MIT-style license. See the `LICENSE` file for details.

---

If you'd like, I can also add example integration code (Spring Boot filter), benchmark scripts, or convert critical operations to Lua for atomicity. Which would you prefer next?
