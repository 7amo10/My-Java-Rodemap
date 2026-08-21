---
tags: [jakarta-ee, servlet, filters, async-servlet, lifecycle, phase-3]
---

# :material-book-open-page-variant: EE8 AppDev — Chapter 13: Servlet Development and Deployment

> **Book:** Java EE 8 Application Development — David R. Heffelfinger (Packt)  
> **Chapter:** 13 — Servlet Development and Deployment  
> **Relevance to Day 05:** Conceptual foundation for Servlet Filters — the container-level equivalent of JAX-RS `ContainerRequestFilter`

---

## :material-information: What is a Servlet?

A **Servlet** is a Java class that extends the capability of a server (typically HTTP). The Jakarta EE standard:

- Base class: `jakarta.servlet.GenericServlet`
- HTTP specialization: `jakarta.servlet.http.HttpServlet`

`HttpServlet` provides overridable methods for each HTTP verb:

| HTTP Request | `HttpServlet` Method |
|-------------|---------------------|
| `GET` | `doGet(HttpServletRequest, HttpServletResponse)` |
| `POST` | `doPost(HttpServletRequest, HttpServletResponse)` |
| `PUT` | `doPut(HttpServletRequest, HttpServletResponse)` |
| `DELETE` | `doDelete(HttpServletRequest, HttpServletResponse)` |
| `HEAD` | `doHead(...)` |
| `OPTIONS` | `doOptions(...)` |

---

## :material-code-tags: Writing a Servlet — `@WebServlet`

No `web.xml` required — just annotate:

```java
@WebServlet(urlPatterns = {"/simpleservlet"})
public class SimpleServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        resp.setContentType("text/html");
        PrintWriter writer = resp.getWriter();
        writer.println("<h2>Hello Servlet World!</h2>");
    }
}
```

---

## :material-form-select: Processing HTML Forms

Extract form parameters with `getParameter()`:

```java
@WebServlet(urlPatterns = {"/formhandler"})
public class FormHandlerServlet extends HttpServlet {

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        String username  = req.getParameter("username");         // single value
        String[] options = req.getParameterValues("preferences"); // multiple values (checkboxes)
        String password  = req.getParameter("password");
        // process...
    }
}
```

---

## :material-arrow-right-bold: Request Forwarding vs Response Redirection

### Request Forwarding — same request, server-side

The request context (attributes, parameters) is preserved. The client sees the same URL.

```java
// Attach data to request:
request.setAttribute("result", computedResult);

// Forward to another servlet:
request.getRequestDispatcher("/confirmationServlet")
       .forward(request, response);

// In destination servlet — read attributes:
Object result = request.getAttribute("result");
```

### Response Redirection — new request, client-side

Instructs the browser to make a new GET request. Request attributes are lost.

```java
String externalUrl = request.getParameter("redirectUrl");
response.sendRedirect(externalUrl);
```

---

## :material-database: Persisting Data Across Requests

### Session Scope (`HttpSession`) — per user

```java
HttpSession session = request.getSession();   // creates if not exists
session.setAttribute("currentUser", user);

// Later request:
User user = (User) session.getAttribute("currentUser");
session.invalidate();   // logout
```

### Application Scope (`ServletContext`) — shared across all users

```java
ServletContext ctx = getServletContext();
ctx.setAttribute("appConfig", config);

// Anywhere in the app:
AppConfig config = (AppConfig) servletContext.getAttribute("appConfig");
```

---

## :material-tune: Servlet Init Parameters — `@WebInitParam`

Pass configuration to a servlet at startup:

```java
@WebServlet(
    name = "ConfiguredServlet",
    urlPatterns = {"/configured"},
    initParams = {
        @WebInitParam(name = "apiBaseUrl", value = "https://api.example.com"),
        @WebInitParam(name = "timeout",    value = "30")
    }
)
public class ConfiguredServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) {
        String apiUrl = getServletConfig().getInitParameter("apiBaseUrl");
        String timeout = getServletConfig().getInitParameter("timeout");
    }
}
```

---

## :material-filter: Servlet Filters — `Filter` + `@WebFilter`

Servlet Filters intercept requests **at the Servlet container level** — before JAX-RS runtime even sees them. They implement `jakarta.servlet.Filter`.

!!! tip "Servlet Filter vs JAX-RS ContainerRequestFilter"
    | | Servlet Filter | JAX-RS ContainerRequestFilter |
    |--|---|---|
    | Level | Servlet container | JAX-RS runtime (inside DispatcherServlet) |
    | Sees | Raw `HttpServletRequest` | JAX-RS `ContainerRequestContext` |
    | Order | `@WebFilter` + `web.xml` ordering | `@Priority` annotation |
    | Stops chain | `filterChain.doFilter()` not called | `requestContext.abortWith(Response)` |

### Implementing a Servlet Filter

```java
@WebFilter(
    filterName = "AuditFilter",
    urlPatterns = {"/*"},                          // all URLs
    initParams = {@WebInitParam(name = "logLevel", value = "DEBUG")}
)
public class AuditFilter implements Filter {
    private FilterConfig filterConfig;

    @Override
    public void init(FilterConfig config) throws ServletException {
        this.filterConfig = config;
        // access init params: config.getInitParameter("logLevel")
    }

    @Override
    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain)
            throws IOException, ServletException {
        // PRE-processing (before servlet):
        filterConfig.getServletContext().log("Request entering filter");
        long start = System.currentTimeMillis();

        // Pass to next filter in chain (or servlet):
        chain.doFilter(req, resp);

        // POST-processing (after servlet):
        long elapsed = System.currentTimeMillis() - start;
        filterConfig.getServletContext().log("Request completed in " + elapsed + "ms");
    }

    @Override
    public void destroy() {
        filterConfig = null;
    }
}
```

---

## :material-bell: Servlet Listeners

React to lifecycle events without modifying servlet code:

| Listener Interface | Events Handled |
|------------------|---------------|
| `ServletContextListener` | Application start/stop (`contextInitialized`, `contextDestroyed`) |
| `ServletContextAttributeListener` | Attributes added/removed/replaced in `ServletContext` |
| `ServletRequestListener` | Request created/destroyed |
| `ServletRequestAttributeListener` | Attributes in request scope |
| `HttpSessionListener` | Session created/destroyed |
| `HttpSessionAttributeListener` | Attributes in session scope |

```java
@WebListener
public class ApplicationLifecycleListener implements ServletContextListener {

    @Override
    public void contextInitialized(ServletContextEvent event) {
        ServletContext ctx = event.getServletContext();
        ctx.log("Application starting — initializing caches");
        // pre-warm caches, load configuration, etc.
    }

    @Override
    public void contextDestroyed(ServletContextEvent event) {
        event.getServletContext().log("Application shutting down — cleanup");
    }
}
```

---

## :material-timer: Asynchronous Servlet Processing

For long-running operations, release the HTTP thread immediately and complete the response asynchronously:

```java
@WebServlet(
    name = "AsyncServlet",
    urlPatterns = {"/async"},
    asyncSupported = true      // MUST enable async explicitly
)
public class AsyncServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        // Start async context — releases HTTP thread back to thread pool
        AsyncContext asyncCtx = req.startAsync();

        // Dispatch long work to a different thread
        asyncCtx.start(() -> {
            try {
                Thread.sleep(5000);   // simulate long computation
                asyncCtx.getResponse().getWriter()
                    .println("Result ready after 5s!");
                asyncCtx.complete();   // signals response is done
            } catch (Exception e) {
                asyncCtx.complete();
            }
        });
        // doGet() returns immediately — HTTP thread is freed
    }
}
```

!!! note "Async Servlet vs ManagedExecutorService"
    In Week 3 (Day 16), you'll use Jakarta EE's `ManagedExecutorService` instead of raw `asyncCtx.start()`. The managed executor respects CDI contexts and transaction propagation.

---

## :material-programmatic-setup: Programmatic Servlet Registration

Register servlets dynamically at startup via `ServletContextListener`:

```java
@WebListener
public class DynamicRegistrationListener implements ServletContextListener {

    @Override
    public void contextInitialized(ServletContextEvent event) {
        ServletContext ctx = event.getServletContext();
        try {
            DynamicServlet servlet = ctx.createServlet(DynamicServlet.class);
            ctx.addServlet("DynamicServlet", servlet)
               .addMapping("/dynamic/*");
        } catch (Exception e) {
            throw new RuntimeException("Failed to register servlet", e);
        }
    }
}
```

---

## :material-http: HTTP/2 Server Push

Proactively push resources to the browser before they're requested:

```java
@Override
protected void doGet(HttpServletRequest req, HttpServletResponse resp) {
    PushBuilder pushBuilder = req.newPushBuilder();   // null if HTTP/1.1
    if (pushBuilder != null) {
        // Push CSS and JavaScript before the browser even parses the HTML
        pushBuilder.path("css/main.css").addHeader("content-type", "text/css").push();
        pushBuilder.path("js/app.js").addHeader("content-type", "application/javascript").push();
    }
    // Then send the HTML response
    resp.getWriter().println("<html>...");
}
```

---

## :material-key: Key Takeaways

1. `@WebServlet(urlPatterns = {"/path"})` registers a servlet without `web.xml`
2. **Servlet Filters** fire at the container level — before JAX-RS sees the request; use for global cross-cutting (CORS, logging, IP filtering)
3. **JAX-RS `ContainerRequestFilter`** fires inside the JAX-RS runtime — better for REST-specific concerns (authentication, content negotiation)
4. `HttpSession` = per-user scope; `ServletContext` = application-wide scope
5. `asyncSupported = true` + `req.startAsync()` releases the HTTP thread for long-running operations
6. `@WebListener` marks a class as a lifecycle observer — no manual registration needed

---

[:octicons-arrow-left-24: Back to Day 05 Index](index.md)
