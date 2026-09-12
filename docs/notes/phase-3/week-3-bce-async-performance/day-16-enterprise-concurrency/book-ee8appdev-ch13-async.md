---
tags: [jakarta-ee, servlet, async-processing, managed-threads, concurrency, phase-3]
---

# :material-book-open-page-variant: EE8 AppDev — Chapter 13: Servlet Development & Async Processing

> **Book:** Java EE 8 Application Development (Packt)  
> **Chapter:** 13 — Servlet Development (focusing on Asynchronous Processing & Async Servlets)

---

## :material-information: Why Asynchronous Processing?

Standard synchronous servlets block an HTTP worker thread for the entire duration of a request. For long-running operations (external API calls, database batch processing, file generation), this causes **thread starvation** — all worker threads wait idle while tasks run, preventing new requests from being processed.

Servlet 3.0 introduced `AsyncContext` to **release the HTTP worker thread** while the actual work continues in a background thread, allowing the worker to serve other requests.

---

## :material-code-braces: Servlet Async Processing API

### Basic Pattern

```java
@WebServlet(
    name = "DiagnosticServlet",
    urlPatterns = {"/api/diagnostics"},
    asyncSupported = true                 // ← MUST enable async support
)
public class DiagnosticServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {

        // Release the HTTP worker thread:
        AsyncContext asyncContext = request.startAsync();
        asyncContext.setTimeout(30_000);   // 30 second timeout

        // Submit work to background thread:
        asyncContext.start(() -> {
            try {
                Thread.sleep(5000);   // Simulated long operation

                PrintWriter out = asyncContext.getResponse().getWriter();
                out.println("{\"status\": \"diagnostics complete\"}");
                out.flush();

                asyncContext.complete();   // MUST call when done — commits response
            } catch (Exception e) {
                asyncContext.complete();
            }
        });

        // Returns immediately — HTTP worker thread is now FREE to handle other requests
    }
}
```

### Key `AsyncContext` Methods

| Method | Purpose |
|--------|---------|
| `request.startAsync()` | Release HTTP worker thread; returns `AsyncContext` |
| `asyncContext.complete()` | Commit response and close the async cycle — MUST be called |
| `asyncContext.dispatch(path)` | Forward to another servlet/JSP after async processing |
| `asyncContext.setTimeout(ms)` | Set max time before container auto-completes with error |
| `asyncContext.getRequest()` | Access the original `HttpServletRequest` |
| `asyncContext.getResponse()` | Access the `HttpServletResponse` to write output |

---

## :material-compare: Async Servlet vs ManagedExecutorService

| Aspect | Async Servlet (`AsyncContext`) | `ManagedExecutorService` |
|--------|-------------------------------|-------------------------|
| Thread used | `asyncContext.start(Runnable)` — container thread | `ManagedExecutorService` — container-governed pool |
| Context propagation | Minimal — no security/CDI propagation | Full — security principal, correlation ID, CDI |
| Lifecycle governance | Container manages timeouts | Container manages pool lifecycle |
| Integration | Servlet-centric | Full Jakarta EE — works in CDI beans, EJBs |
| **Recommended for** | Simple async responses | Enterprise workloads needing full context |

!!! important "Prefer `ManagedExecutorService` for enterprise code"
    For any business logic that needs access to `SecurityContext`, `@PersistenceContext`, CDI injection, or transactional resources, always use `ManagedExecutorService`. `AsyncContext.start()` provides no such guarantees.

---

## :material-web: Servlet Core Concepts Reference

### Request Scopes

```java
// Request scope — lives for one HTTP request:
request.setAttribute("result", diagnosticReport);

// Session scope — lives for the user's session:
request.getSession().setAttribute("user", authenticatedUser);

// Application scope — lives for entire app lifetime:
getServletContext().setAttribute("config", appConfig);
```

### Servlet Filters — `@WebFilter`

```java
@WebFilter(urlPatterns = {"/api/secure/*"})
public class SecurityFilter implements Filter {

    @Override
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
            throws IOException, ServletException {

        HttpServletRequest httpReq = (HttpServletRequest) req;

        String token = httpReq.getHeader("Authorization");
        if (token == null || !token.startsWith("Bearer ")) {
            ((HttpServletResponse) res).setStatus(401);
            return;
        }

        chain.doFilter(req, res);   // Pass to next filter or resource
    }
}
```

### Programmatic Servlet Registration

```java
@WebListener
public class AppInitListener implements ServletContextListener {

    @Override
    public void contextInitialized(ServletContextEvent sce) {
        ServletContext ctx = sce.getServletContext();

        // Register servlet programmatically:
        MyServlet servlet = ctx.createServlet(MyServlet.class);
        ServletRegistration.Dynamic reg = ctx.addServlet("myServlet", servlet);
        reg.addMapping("/api/custom/*");
        reg.setAsyncSupported(true);
    }
}
```

### HTTP/2 Server Push (Servlet 4.0+)

```java
protected void doGet(HttpServletRequest request, HttpServletResponse response) {
    PushBuilder pushBuilder = request.newPushBuilder();
    if (pushBuilder != null) {
        // Proactively send static assets before browser requests them:
        pushBuilder.path("css/dashboard.css")
                   .addHeader("content-type", "text/css")
                   .push();
        pushBuilder.path("js/cluster-monitor.js")
                   .addHeader("content-type", "application/javascript")
                   .push();
    }
    // Now render the main HTML:
    response.getWriter().println("<html>...</html>");
}
```

---

## :material-key: Key Takeaways — Ch13 Async Servlets

1. **`asyncSupported = true`** — required on `@WebServlet` (or `web.xml`) before calling `startAsync()`
2. **`asyncContext.complete()`** — MUST always be called (even in catch blocks) to commit the response; not calling it leaks resources
3. **`asyncContext.setTimeout(ms)`** — prevents indefinitely hanging connections; container auto-completes after timeout
4. **`AsyncContext.start(Runnable)`** — runs the Runnable on a container thread — but WITHOUT security/CDI context propagation → use `ManagedExecutorService` for business logic
5. **Filters work with async** — `@WebFilter` can set `asyncSupported = true` to intercept async dispatches too
6. **Prefer `ManagedExecutorService`** for any enterprise code that needs JPA, security, or CDI context

---

[:octicons-arrow-left-24: Back to Day 16 Index](index.md)
