---
tags: [jakarta-ee, json-b, json-p, exception-mapping, rfc-7807, lab, phase-3]
---

# :material-flask: Day 04 Lab Guide — JSON Processing & Exception Architecture

> **Module:** Week 1 — CDI 4.0, JAX-RS 3.1 & REST Services  
> **Lab Repo:** [:material-github: 7amo10/JavaEE-Labs — Week-1-CDI-JAXRS-REST](https://github.com/7amo10/JavaEE-Labs/tree/main/Week-1-CDI-JAXRS-REST)

---

## :material-target: Lab Objective

Master **Jakarta JSON Binding (JSON-B 3.0)**, **Jakarta JSON Processing (JSON-P 2.1)**, and JAX-RS **`ExceptionMapper`** providers returning standardized **RFC-7807** problem details payloads.

---

## :material-list-box: Key Concepts Covered

- **JSON-B 3.0:** `@JsonbProperty`, `@JsonbDateFormat`, `@JsonbTransient`, `@JsonbTypeAdapter`, `@JsonbCreator`
- **JSON-P 2.1:** `Json.createObjectBuilder()`, `JsonArrayBuilder`, dynamic DOM tree construction
- **`ExceptionMapper<T>` + `@Provider`:** Decoupling exception throwing from HTTP status mapping
- **RFC-7807 Problem Details:** `application/problem+json` standardized error schema

---

## :material-code-braces: Component Architecture

Package: `com.ee.lab.json`

| Component | Purpose |
|-----------|---------|
| `ProblemDetails` | RFC-7807 record: `type`, `title`, `status`, `detail`, `instance` |
| `ResourceNotFoundException` | Domain exception → HTTP 404 |
| `InvalidPayloadException` | Domain exception → HTTP 400 |
| `DuplicateResourceException` | Domain exception → HTTP 409 |
| `ResourceNotFoundExceptionMapper` | `@Provider ExceptionMapper<ResourceNotFoundException>` |
| `InvalidPayloadExceptionMapper` | `@Provider ExceptionMapper<InvalidPayloadException>` |
| `DuplicateResourceExceptionMapper` | `@Provider ExceptionMapper<DuplicateResourceException>` |
| `GenericExceptionMapper` | `@Provider ExceptionMapper<Throwable>` — safety net → 500 |
| `AnalysisJob` | Domain model with JSON-B customization annotations |
| `JobStatusAdapter` | `JsonbAdapter<JobStatus, String>` for enum wire mapping |
| `AnalysisJobResource` | REST endpoints + JSON-P dynamic AST endpoint |
| `AppRunner` | Automated test harness |

---

## :material-stairs: Key Implementation Snippets

### RFC-7807 Problem Details Model

```java
public record ProblemDetails(
    @JsonbProperty("type")    String type,
    @JsonbProperty("title")   String title,
    @JsonbProperty("status")  int status,
    @JsonbProperty("detail")  String detail,
    @JsonbProperty("instance") String instance
) {}
```

### Exception Mapper with RFC-7807

```java
@Provider
public class ResourceNotFoundExceptionMapper
        implements ExceptionMapper<ResourceNotFoundException> {

    @Override
    public Response toResponse(ResourceNotFoundException e) {
        ProblemDetails problem = new ProblemDetails(
            "https://problems.example.com/resource-not-found",
            "Resource Not Found",
            404,
            e.getMessage(),
            "/api/v1/jobs/" + e.getResourceId()
        );
        return Response.status(404)
            .type("application/problem+json")
            .entity(problem)
            .build();
    }
}
```

### JSON-B Customized Model

```java
public class AnalysisJob {
    @JsonbProperty("job_id")           private String id;
    @JsonbDateFormat("yyyy-MM-dd'T'HH:mm:ss")
    @JsonbProperty("created_at")       private LocalDateTime createdAt;
    @JsonbTransient                    private String internalSecuritySignature;
    @JsonbTypeAdapter(JobStatusAdapter.class)
                                       private JobStatus status;
    // getters/setters...
}
```

### JSON-P Dynamic AST Construction

```java
@GET @Path("{id}/ast")
@Produces(MediaType.APPLICATION_JSON)
public Response getBytecodeAst(@PathParam("id") String id) {
    JsonObject ast = Json.createObjectBuilder()
        .add("jobId", id)
        .add("opcodes", Json.createArrayBuilder()
            .add(Json.createObjectBuilder()
                .add("opcode", "INVOKEVIRTUAL")
                .add("count", 42)
                .build())
            .add(Json.createObjectBuilder()
                .add("opcode", "GETFIELD")
                .add("count", 18)
                .build())
            .build())
        .add("generatedAt", Instant.now().toString())
        .build();

    return Response.ok(ast.toString()).build();
}
```

---

## :material-check-all: Verification Checklist

| # | Request | Expected |
|---|---------|----------|
| 1 | Valid `POST /api/v1/jobs` | HTTP 201, JSON-B custom keys (`job_id`, `created_at`), `Location` header |
| 2 | Duplicate `POST /api/v1/jobs` | HTTP 409, `Content-Type: application/problem+json`, RFC-7807 body |
| 3 | Invalid `POST /api/v1/jobs` (missing required field) | HTTP 400, field validation details in `detail` field |
| 4 | `GET /api/v1/jobs/nonexistent` | HTTP 404, RFC-7807 problem details |
| 5 | `GET /api/v1/jobs/{id}` (existing) | `internalSecuritySignature` field **completely absent** from JSON |
| 6 | `GET /api/v1/jobs/{id}/ast` | HTTP 200, dynamic JSON-P bytecode AST object |

---

[:octicons-arrow-left-24: Back to Day 04 Index](index.md) | [:material-github: View Lab Solution](https://github.com/7amo10/JavaEE-Labs/tree/main/Week-1-CDI-JAXRS-REST)
