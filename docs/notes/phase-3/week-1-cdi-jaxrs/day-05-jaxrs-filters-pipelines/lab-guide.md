---
tags: [jakarta-ee, jax-rs, filters, security, lab, phase-3]
---

# :material-flask: Day 05 Lab Guide — JAX-RS 3.1 Filters & Pipelines

> **Module:** Week 1 — CDI 4.0, JAX-RS 3.1 & REST Services  
> **Lab Repo:** [:material-github: 7amo10/JavaEE-Labs — Week-1-CDI-JAXRS-REST](https://github.com/7amo10/JavaEE-Labs/tree/main/Week-1-CDI-JAXRS-REST)

---

## :material-target: Lab Objective

Master the **JAX-RS 3.1 filter pipeline architecture**: implement global `@PreMatching` request logging, name-bound authentication filters with short-circuit request aborting (`abortWith`), and dynamic response header enrichment.

---

## :material-list-box: Key Concepts Covered

| Concept | Key Type |
|---------|----------|
| Pre-matching global filter | `@PreMatching ContainerRequestFilter` |
| Post-matching selective filter | `@NameBinding` + `ContainerRequestFilter` |
| Filter execution priority | `@Priority(Priorities.AUTHENTICATION)` |
| Short-circuit abort | `requestContext.abortWith(Response.status(401).build())` |
| Response enrichment | `ContainerResponseFilter` |
| Correlation ID propagation | `X-Correlation-Id` UUID header |

---

## :material-code-braces: Component Architecture

Package: `com.ee.lab.filters`

| Component | Role |
|-----------|------|
| `@Authenticated` | Custom `@NameBinding` annotation — marks protected resources |
| `PreMatchingLoggingFilter` | `@PreMatching` — fires before routing; assigns `X-Correlation-Id` |
| `BearerTokenAuthFilter` | Name-bound `@Authenticated` — validates Bearer token; `abortWith(401)` on failure |
| `ResponseEnrichmentFilter` | `ContainerResponseFilter` — calculates request duration; injects `X-Execution-Time-Millis`, `X-Correlation-Id`, `X-Security-Policy` |
| `PublicDiagnosticsResource` | `/diagnostics/ping` — open endpoint, no auth |
| `ProtectedMetricsResource` | `/metrics/secure` — `@Authenticated` protected endpoint |
| `AppRunner` | Automated test harness |

---

## :material-stairs: Key Implementation Snippets

### Step 1: Define `@NameBinding` Annotation

```java
@NameBinding
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
public @interface Authenticated {}
```

### Step 2: `@PreMatching` Correlation ID Filter

```java
@PreMatching
@Provider
@Priority(Priorities.HEADER_DECORATOR)
public class PreMatchingLoggingFilter implements ContainerRequestFilter {

    @Override
    public void filter(ContainerRequestContext requestContext) {
        String correlationId = requestContext.getHeaderString("X-Correlation-Id");
        if (correlationId == null || correlationId.isBlank()) {
            correlationId = UUID.randomUUID().toString();
        }
        // Store for later retrieval by response filter
        requestContext.setProperty("correlationId", correlationId);
        requestContext.setProperty("requestStartTime", System.currentTimeMillis());

        System.out.printf("[REQUEST] %s %s | Correlation-Id: %s%n",
            requestContext.getMethod(),
            requestContext.getUriInfo().getRequestUri(),
            correlationId);
    }
}
```

### Step 3: Bearer Token Auth Filter (Name-Bound)

```java
@Authenticated
@Provider
@Priority(Priorities.AUTHENTICATION)
public class BearerTokenAuthFilter implements ContainerRequestFilter {

    private static final Set<String> VALID_TOKENS = Set.of("secret-token-123");

    @Override
    public void filter(ContainerRequestContext requestContext) {
        String authHeader = requestContext.getHeaderString(HttpHeaders.AUTHORIZATION);
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            requestContext.abortWith(
                Response.status(Response.Status.UNAUTHORIZED)
                    .entity("{\"error\":\"Missing Authorization header\"}")
                    .type(MediaType.APPLICATION_JSON)
                    .build()
            );
            return;
        }
        String token = authHeader.substring("Bearer ".length()).trim();
        if (!VALID_TOKENS.contains(token)) {
            requestContext.abortWith(
                Response.status(Response.Status.UNAUTHORIZED)
                    .entity("{\"error\":\"Invalid Bearer token\"}")
                    .type(MediaType.APPLICATION_JSON)
                    .build()
            );
        }
    }
}
```

### Step 4: Response Enrichment Filter

```java
@Provider
public class ResponseEnrichmentFilter implements ContainerResponseFilter {

    @Override
    public void filter(ContainerRequestContext req, ContainerResponseContext resp) {
        String correlationId = (String) req.getProperty("correlationId");
        Long startTime = (Long) req.getProperty("requestStartTime");

        if (correlationId != null) {
            resp.getHeaders().add("X-Correlation-Id", correlationId);
        }
        if (startTime != null) {
            long elapsed = System.currentTimeMillis() - startTime;
            resp.getHeaders().add("X-Execution-Time-Millis", elapsed);
        }
        resp.getHeaders().add("X-Security-Policy", "strict-transport-security");
    }
}
```

### Step 5: Protected Resource with `@Authenticated`

```java
@Path("/metrics")
@Authenticated
@Produces(MediaType.APPLICATION_JSON)
public class ProtectedMetricsResource {

    @GET @Path("/secure")
    public Response getSecureMetrics() {
        Map<String, Object> metrics = Map.of(
            "heapUsedMb", Runtime.getRuntime().totalMemory() / (1024 * 1024),
            "activeThreads", Thread.activeCount()
        );
        return Response.ok(metrics).build();
    }
}
```

---

## :material-check-all: Verification Checklist

| # | Request | Expected Behavior |
|---|---------|------------------|
| 1 | `GET /diagnostics/ping` | HTTP 200, `X-Correlation-Id` + `X-Execution-Time-Millis` headers present, **no auth required** |
| 2 | `GET /metrics/secure` (no `Authorization` header) | HTTP 401 — `BearerTokenAuthFilter.abortWith()` fires |
| 3 | `GET /metrics/secure` with `Authorization: Bearer wrong-token` | HTTP 401 — invalid token rejected |
| 4 | `GET /metrics/secure` with `Authorization: Bearer secret-token-123` | HTTP 200 with live runtime metrics + enriched headers |

!!! tip "Key insight"
    Even when `abortWith()` fires (cases 2 & 3), the `ResponseEnrichmentFilter` **still runs** and attaches headers to the 401 response. The resource method itself is **never invoked**.

---

[:octicons-arrow-left-24: Back to Day 05 Index](index.md) | [:material-github: View Lab Solution](https://github.com/7amo10/JavaEE-Labs/tree/main/Week-1-CDI-JAXRS-REST)
