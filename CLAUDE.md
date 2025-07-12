# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Resilience4j is a lightweight fault tolerance library designed for functional programming in Java 17+. It provides higher-order functions (decorators) to enhance any functional interface with resilience patterns like Circuit Breaker, Rate Limiter, Retry, Bulkhead, Time Limiter, and Cache.

## Essential Development Commands

### Building
```bash
./gradlew build              # Full build with tests
./gradlew assemble           # Build without tests
./gradlew clean build        # Clean build
./gradlew :resilience4j-circuitbreaker:build  # Build specific module
```

### Testing
```bash
./gradlew test               # Run all tests
./gradlew :resilience4j-circuitbreaker:test  # Test specific module
./gradlew check              # Run all verification tasks
./gradlew jacocoTestReport   # Generate coverage report
./gradlew testCodeCoverageReport  # Aggregated coverage report
```

### Code Quality
```bash
./gradlew sonarqube          # Run SonarQube analysis
./gradlew jmh                # Run performance benchmarks
```

### Publishing
```bash
./gradlew publishToMavenLocal       # Publish to local Maven repo
./gradlew publishToSonatype         # Publish to Sonatype (requires credentials)
```

## High-Level Architecture

### Core Design Patterns

1. **Decorator Pattern**: All resilience patterns implement decorators for functional interfaces:
   - `Supplier<T>`, `Function<T,R>`, `Runnable`, `Callable<T>`, `Consumer<T>`
   - Support for async operations via `CompletionStage` and `Future`

2. **Registry Pattern**: Each resilience pattern has a registry for managing instances:
   - `CircuitBreakerRegistry`, `RetryRegistry`, `BulkheadRegistry`, etc.
   - Provides configuration management, instance creation, and lifecycle management

3. **Event System**: Observable event streams for monitoring:
   - Each pattern publishes events (success, error, state transitions)
   - `EventPublisher<T>` and `EventConsumer<T>` interfaces
   - Thread-safe event processing using `CopyOnWriteArraySet`

4. **State Machine**: Circuit Breaker uses finite state machine:
   - States: CLOSED, OPEN, HALF_OPEN, DISABLED, FORCED_OPEN, METRICS_ONLY
   - Managed by `CircuitBreakerStateMachine`

### Module Structure

```
resilience4j-core/           # Core abstractions and utilities
├── resilience4j-circuitbreaker/   # Circuit breaker implementation
├── resilience4j-retry/            # Retry pattern
├── resilience4j-bulkhead/         # Bulkhead pattern (semaphore & thread pool)
├── resilience4j-ratelimiter/      # Rate limiting
├── resilience4j-timelimiter/      # Time limiting
├── resilience4j-cache/            # Caching
└── resilience4j-all/              # Decorators composition

Framework integrations:
├── resilience4j-spring*/          # Spring Framework integration
├── resilience4j-reactor/          # Project Reactor support
├── resilience4j-rxjava*/          # RxJava support
├── resilience4j-kotlin/           # Kotlin extensions
└── resilience4j-micrometer/       # Metrics integration
```

### Key Abstractions

1. **Registry<E, C>**: Generic registry interface for managing instances
2. **IntervalFunction**: Configurable backoff strategies (fixed, exponential, random)
3. **Decorators**: Fluent API for composing multiple resilience patterns
4. **EventProcessor**: Manages event distribution to consumers

### Configuration Architecture

- Each pattern has immutable configuration classes (e.g., `CircuitBreakerConfig`)
- Builder pattern for configuration creation
- Configuration inheritance: default → named → instance
- Thread-safe configuration management

## Important Development Guidelines

1. **Backward Compatibility**: This is a library used by many projects. Always maintain backward compatibility.

2. **Thread Safety**: All resilience patterns must be thread-safe. Use atomic operations and concurrent data structures.

3. **Testing**: Tests automatically retry up to 3 times. Write comprehensive tests for new features.

4. **Commit Format**: Use format `Issue #XXX: Fixed/Added description`

5. **Module Independence**: Each resilience pattern module should be independently usable.

6. **Performance**: Run JMH benchmarks for performance-critical changes.

7. **Documentation**: Update Javadoc for all public APIs.

## Common Tasks

### Adding a New Resilience Pattern
1. Create new module `resilience4j-<pattern>`
2. Implement core pattern logic extending base abstractions
3. Create `<Pattern>Config` and `<Pattern>ConfigBuilder`
4. Implement `<Pattern>Registry` extending `AbstractRegistry`
5. Add decorator methods for functional interfaces
6. Implement event types extending base events
7. Add integration tests and documentation

### Modifying Existing Patterns
1. Check impact on public API (maintain compatibility)
2. Update both unit and integration tests
3. Run performance benchmarks if touching core logic
4. Update documentation and examples

### Framework Integration
1. Follow existing integration patterns (see spring/reactor modules)
2. Provide both programmatic and declarative (annotation) support
3. Integrate with framework's lifecycle and configuration
4. Add framework-specific tests and documentation