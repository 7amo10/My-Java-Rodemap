---
tags: [jakarta-ee, jax-rs, rest, lab, phase-3]
---

# :material-flask: Day 03 Lab Guide — JAX-RS 3.1 REST Architecture

> **Module:** Week 1 — CDI 4.0, JAX-RS 3.1 & REST Services  
> **Lab Repo:** [:material-github: 7amo10/JavaEE-Labs — Week-1-CDI-JAXRS-REST](https://github.com/7amo10/JavaEE-Labs/tree/main/Week-1-CDI-JAXRS-REST)

---

## :material-target: Lab Objective

Build a complete JAX-RS 3.1 REST resource using **embedded Jersey** and **Grizzly HTTP server**, implementing standard CRUD semantics with proper HTTP status codes, URI template parameter binding, content negotiation, and `Response` object construction.

---

## :material-list-box: Key Concepts Covered

| Concept | Annotation / API |
|---------|-----------------|
| Resource URI template | `@Path("/telemetry")`, `@Path("{id}")` |
| HTTP verb mapping | `@GET`, `@POST`, `@PUT`, `@DELETE` |
| URI path variables | `@PathParam("id")` |
| Query string params | `@QueryParam("limit")` + `@DefaultValue("10")` |
| HTTP request headers | `@HeaderParam("X-Client-Id")` |
| Content negotiation | `@Produces`, `@Consumes` |
| Response construction | `Response.ok()`, `.created()`, `.noContent()`, `.status(404)` |

---

## :material-code-braces: Component Architecture

Package: `com.ee.lab.jaxrs`

| Component | Responsibility |
|-----------|---------------|
| `TelemetryReport` | Domain model — serialized/deserialized via JSON-B automatically |
| `TelemetryRepository` | Thread-safe `ConcurrentHashMap<String, TelemetryReport>` store |
| `TelemetryResource` | JAX-RS 3.1 resource exposing full CRUD at `/api/v1/telemetry` |
| `RestApplication` | `Application` subclass with `@ApplicationPath("/api/v1")` |
| `AppRunner` | Starts embedded Grizzly server; runs automated test calls via Java `HttpClient` |

---

## :material-stairs: Key Implementation Snippets

### Resource Class Skeleton

```java
@Path("/telemetry")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
@RequestScoped
public class TelemetryResource {

    @Inject TelemetryRepository repo;
    @Context UriInfo uriInfo;

    @GET
    public Response getAll(@QueryParam("limit") @DefaultValue("10") int limit) {
        List<TelemetryReport> reports = repo.findAll(limit);
        return Response.ok(reports).build();  // 200
    }

    @POST
    public Response create(TelemetryReport report) {
        repo.save(report);
        URI location = uriInfo.getAbsolutePathBuilder()
            .path(report.getId()).build();
        return Response.created(location).build();  // 201 + Location header
    }

    @GET @Path("{id}")
    public Response getById(
            @PathParam("id") String id,
            @HeaderParam("X-Client-Id") String clientId) {
        return repo.findById(id)
            .map(r -> Response.ok(r).build())         // 200
            .orElse(Response.status(404).build());    // 404
    }

    @PUT @Path("{id}")
    public Response update(@PathParam("id") String id, TelemetryReport report) {
        repo.update(id, report);
        return Response.ok(report).build();           // 200
    }

    @GET @Path("{id}/summary")
    @Produces(MediaType.TEXT_PLAIN)
    public Response getSummary(@PathParam("id") String id) {
        return repo.findById(id)
            .map(r -> Response.ok(r.toTextSummary()).build())
            .orElse(Response.status(404).build());
    }

    @DELETE @Path("{id}")
    public Response delete(@PathParam("id") String id) {
        repo.delete(id);
        return Response.noContent().build();          // 204
    }
}
```

### Embedded Server Startup

```java
URI baseUri = UriBuilder.fromUri("http://localhost/").port(8080).build();
ResourceConfig config = new ResourceConfig()
    .register(TelemetryResource.class)
    .register(JacksonFeature.class);  // or JSON-B provider
HttpServer server = GrizzlyHttpServerFactory.createHttpServer(baseUri, config);
```

---

## :material-check-all: Verification Checklist

| # | Request | Expected |
|---|---------|----------|
| 1 | `GET /api/v1/telemetry?limit=5` | HTTP 200, JSON array |
| 2 | `POST /api/v1/telemetry` body: `{"id":"TEL-101","metric":"heap"}` | HTTP 201, `Location: .../telemetry/TEL-101` |
| 3 | `GET /api/v1/telemetry/TEL-101` with `X-Client-Id: test` | HTTP 200, JSON report |
| 4 | `PUT /api/v1/telemetry/TEL-101` updated body | HTTP 200, updated JSON |
| 5 | `GET /api/v1/telemetry/TEL-101/summary` with `Accept: text/plain` | HTTP 200, plain text |
| 6 | `DELETE /api/v1/telemetry/TEL-101` | HTTP 204, empty body |
| 7 | `GET /api/v1/telemetry/TEL-101` (after delete) | HTTP 404 |

---

[:octicons-arrow-left-24: Back to Day 03 Index](index.md) | [:material-github: View Lab Solution](https://github.com/7amo10/JavaEE-Labs/tree/main/Week-1-CDI-JAXRS-REST)
