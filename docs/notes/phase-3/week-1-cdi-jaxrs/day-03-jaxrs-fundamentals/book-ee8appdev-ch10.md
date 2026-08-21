---
tags: [jakarta-ee, jax-rs, rest, http-verbs, parameters, sse, phase-3]
---

# :material-book-open-page-variant: EE8 AppDev — Chapter 10: RESTful Web Services with JAX-RS

> **Book:** Java EE 8 Application Development — David R. Heffelfinger (Packt)  
> **Chapter:** 10 — RESTful Web Services with JAX-RS  
> **Cross-referenced:** Day 04 (ExceptionMapper section) · Day 05 (Filters section)

---

## :material-information: Introduction to RESTful Web Services

RESTful web services are **flexible and lightweight**, typically producing/consuming JSON or XML. JAX-RS defines the Jakarta EE standard for building REST services using **annotation-driven** POJO classes — zero XML required.

HTTP methods supported by JAX-RS:

| Annotation | HTTP Method | Conventional Use |
|-----------|-------------|-----------------|
| `@GET` | GET | Retrieve an existing resource (safe + idempotent) |
| `@POST` | POST | Update an existing resource |
| `@PUT` | PUT | Create a new resource |
| `@DELETE` | DELETE | Delete an existing resource |

!!! warning "EE8 book HTTP semantics"
    The EE8 AppDev book lists POST=update, PUT=create — which is the opposite of REST conventions in most modern APIs. In practice (and per RFC 7231): **POST=create, PUT=full replace**. Follow RFC 7231 in your labs.

---

## :material-api: Developing a Simple RESTful Web Service

### Resource Class with `@Path`

`@Path` defines the URI for the resource. HTTP verb annotations decorate methods:

```java
@Path("customer")
public class CustomerResource {

    @GET
    @Produces("text/xml")
    public String getCustomer() {
        return "<customer><id>123</id><firstName>Ahmed</firstName></customer>";
    }

    @PUT
    @Consumes("text/xml")
    public void createCustomer(String customerXML) {
        System.out.println("Creating customer: " + customerXML);
    }

    @POST
    @Consumes(MediaType.TEXT_XML)
    public void updateCustomer(String customerXML) {
        System.out.println("Updating customer: " + customerXML);
    }

    @DELETE
    @Consumes("text/xml")
    public void deleteCustomer(String customerXML) {
        System.out.println("Deleting customer: " + customerXML);
    }
}
```

### Configuring the Application Path — `@ApplicationPath`

```java
@ApplicationPath("resources")   // base URI = /appName/resources/
public class JaxRsConfig extends Application {}
```

This is the portable way — no `web.xml` needed.

---

## :material-xml: JAXB — Automatic Java ↔ XML Conversion

Decorate entities with `@XmlRootElement` for transparent JAXB marshaling:

```java
@XmlRootElement
public class Customer implements Serializable {
    private Long id;
    private String firstName;
    private String lastName;
    // constructors, getters, setters
}
```

Method signatures change from `String` to `Customer` — JAX-RS runtime handles serialization automatically:

```java
@GET @Produces("text/xml")
public Customer getCustomer() {
    return new Customer(123L, "Ahmed", "Ashour");
}

@PUT @Consumes("text/xml")
public void createCustomer(Customer customer) {
    // customer is fully populated by JAX-RS
}
```

---

## :material-parameter: Query Parameters — `@QueryParam`

Extract URL query string values:

```java
@GET @Produces("text/xml")
public Customer getCustomer(@QueryParam("id") Long id) {
    return repo.findById(id);
}

// URL: GET /resources/customer?id=123
```

### Via JAX-RS Client API

```java
Client client = ClientBuilder.newClient();
Customer customer = client
    .target("http://localhost:8080/app/resources/customer")
    .queryParam("id", 1L)
    .request()
    .get(Customer.class);
```

---

## :material-code-braces: Path Parameters — `@PathParam`

Embed variable segments in the URI template and extract via `@PathParam`:

```java
@GET
@Produces("text/xml")
@Path("{id}/")             // URI template: /customer/123/
public Customer getCustomer(@PathParam("id") Long id) {
    return repo.findById(id);
}
```

### Via Client API

```java
Customer c = client
    .target("http://localhost:8080/app/resources/customer")
    .path("{id}")
    .resolveTemplate("id", 1L)
    .request()
    .get(Customer.class);
```

### All Parameter Annotations (Full Table)

| Annotation | Source | Example |
|-----------|--------|---------|
| `@PathParam` | URI template variable | `/users/{id}` |
| `@QueryParam` | URL query string | `?page=2&limit=10` |
| `@HeaderParam` | HTTP request header | `X-Client-Id: dashboard` |
| `@FormParam` | HTML form POST body | `username=ahmed` |
| `@CookieParam` | HTTP Cookie header | `sessionId=abc123` |
| `@BeanParam` | Bundles multiple params into a POJO | — |
| `@DefaultValue` | Fallback value | `@DefaultValue("10") int limit` |

---

## :material-handshake: Content Negotiation — `@Produces` and `@Consumes`

JAX-RS matches client `Accept` and `Content-Type` headers to `@Produces`/`@Consumes`:

```java
@GET @Path("{id}/summary")
@Produces({MediaType.APPLICATION_JSON, MediaType.TEXT_PLAIN})
public Response getSummary(@PathParam("id") String id) {
    TelemetryReport report = repo.findById(id).orElseThrow();
    // JAX-RS picks JSON or plain text based on client's Accept header
    return Response.ok(report.toTextSummary()).build();
}
```

---

## :material-hammer-wrench: Response Builder API

The `Response` class provides a fluent builder for complete HTTP responses:

```java
// 200 OK with entity
Response.ok(entity).build()

// 201 Created with Location header
URI location = uriInfo.getAbsolutePathBuilder().path(id).build();
Response.created(location).build()

// 204 No Content
Response.noContent().build()

// 404 Not Found with entity
Response.status(Response.Status.NOT_FOUND)
        .entity(new ErrorBody("Resource not found"))
        .type(MediaType.APPLICATION_JSON)
        .build()

// Custom status code
Response.status(409).entity(conflictBody).build()
```

---

## :material-client-service: JAX-RS Client API

A complete programmatic HTTP client without raw sockets:

```java
public void insertCustomer(Customer customer) {
    Client client = ClientBuilder.newClient();
    client.target("http://localhost:8080/app/resources/customer")
          .request()
          .put(Entity.entity(customer, "text/xml"), Customer.class);
}

// Building fluent request chain:
Response response = client
    .target(baseUrl)
    .path("customer/{id}")
    .resolveTemplate("id", 123L)
    .request(MediaType.APPLICATION_JSON)
    .header("X-Client-Id", "dashboard")
    .get();

int status = response.getStatus();
Customer result = response.readEntity(Customer.class);
```

---

## :material-broadcast: Server-Sent Events (SSE)

JAX-RS 2.1 (Jakarta EE 8) introduced standard SSE support for **server-push streaming**:

```java
@Path("serversentevents")
public class SseResource {

    @GET
    @Produces(MediaType.SERVER_SENT_EVENTS)
    public void streamEvents(@Context SseEventSink sink, @Context Sse sse) {
        Executor executor = Executors.newSingleThreadExecutor();
        executor.execute(() -> {
            for (float value : getStockValues()) {
                try {
                    TimeUnit.SECONDS.sleep(5);
                    OutboundSseEvent event = sse.newEventBuilder()
                        .name("stock-ticker")
                        .data(String.class, String.format("%.2f", value))
                        .build();
                    sink.send(event);   // push to connected client
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
            sink.close();
        });
    }
}
```

### JavaScript Client for SSE

```html
<script>
function subscribeToStock() {
    var source = new EventSource("/app/resources/serversentevents");
    source.addEventListener("stock-ticker", function(event) {
        document.getElementById("price").textContent = event.data;
    }, false);
}
</script>
```

!!! note "SSE Deep Dive in Week 3"
    Full SSE coverage (broadcast channels, `SseBroadcaster`, live JVM metric streaming) is in Week 3 — Day 17.

---

## :material-key: Key Takeaways

1. `@ApplicationPath` on an `Application` subclass = zero-XML JAX-RS configuration
2. `@Produces` / `@Consumes` enable content negotiation — JAX-RS picks the best match from client `Accept` header
3. `@PathParam` extracts from URI template; `@QueryParam` from query string; `@HeaderParam` from request headers
4. `Response` builder gives complete control over status code, headers, and entity body
5. JAX-RS Client API provides a fluent, type-safe HTTP client without raw `HttpURLConnection`
6. SSE (`SseEventSink`) enables **server-initiated push** — client opens one long-lived HTTP connection and receives streamed events

---

[:octicons-arrow-left-24: Back to Day 03 Index](index.md) | [:octicons-arrow-right-24: ExceptionMapper section used in Day 04](../day-04-json-exception-mapping/book-ee8appdev-ch10-exception-mapper.md)
