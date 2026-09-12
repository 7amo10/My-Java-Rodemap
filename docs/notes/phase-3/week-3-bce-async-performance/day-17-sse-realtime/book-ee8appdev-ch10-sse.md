---
tags: [jakarta-ee, sse, jax-rs, server-sent-events, broadcaster, phase-3]
---

# :material-book-open-page-variant: EE8 AppDev — Chapter 10: JAX-RS Server-Sent Events (SSE)

> **Book:** Java EE 8 Application Development (Packt)  
> **Chapter:** 10 — RESTful Web Services (SSE section)

---

## :material-information: SSE Overview

Server-Sent Events (SSE) allow a JAX-RS resource to **push data to HTTP clients over a persistent connection** without the client polling. The server sends events as plain text (`text/event-stream`) over a long-lived HTTP connection. Clients auto-reconnect on disconnect using the built-in `retry` mechanism.

---

## :material-compare: SSE vs WebSockets vs HTTP Polling

| Dimension | Server-Sent Events (SSE) | WebSockets | HTTP Polling |
|-----------|--------------------------|-----------|-------------|
| **Protocol** | Standard HTTP/1.1 or HTTP/2 | Full-duplex WS/WSS | Repeated HTTP |
| **Directionality** | Unidirectional (Server → Client) | Bidirectional (Client ↔ Server) | Client-initiated |
| **Content-Type** | `text/event-stream` | Binary / Text Frames | `application/json` |
| **Auto-Reconnect** | Built-in (`retry:` field) | Custom client logic | Client polling timer |
| **Firewall Traversal** | Works on port 80/443 | Requires proxy WS upgrade | Standard HTTP |
| **Resource Cost** | Low (single long-lived conn) | Moderate (full-duplex state) | High (repeated handshakes) |
| **Use Case** | Live dashboards, notifications | Chat, gaming, bidirectional | Simple status checks |

---

## :material-code-braces: Core JAX-RS SSE Components

### 1. `SseEventSink` — One Client Connection

```java
@GET
@Path("/cluster/stream")
@Produces(MediaType.SERVER_SENT_EVENTS)
public void subscribe(@Context SseEventSink sink,
                      @Context Sse sse) {
    // sink = this client's output channel
    // Method must return void
    // Connection stays open until sink.close() or client disconnects
}
```

### 2. `Sse` — Event Factory

```java
@Context Sse sse;

// Build a structured event:
OutboundSseEvent event = sse.newEventBuilder()
    .name("cluster-alert")             // event: cluster-alert
    .id("evt-" + UUID.randomUUID())    // id: evt-abc123
    .data(TelemetryRecord.class, record)  // data: (JSON-B serialized)
    .reconnectDelay(3000)              // retry: 3000
    .comment("high CPU detected")      // : high CPU detected
    .build();

sink.send(event);
```

### 3. `OutboundSseEvent` — The SSE Wire Format

Each call to `sink.send(event)` writes over the wire:

```
event: cluster-alert
id: evt-abc123
retry: 3000
: high CPU detected
data: {"nodeId":42,"cpuUsage":98.5,"timestamp":"2026-09-12T15:00:00Z"}

```

!!! important "Blank line terminates each event"
    Events are separated by a **blank line (`\n\n`)**. The client's `EventSource` JS API parses this automatically.

---

## :material-broadcast: `SseBroadcaster` — Multicast Fan-Out

`SseBroadcaster` maintains a registry of connected client sinks and fans out events to all simultaneously:

```java
@Path("/cluster")
@ApplicationScoped
public class ClusterStreamResource {

    @Context Sse sse;

    private SseBroadcaster broadcaster;

    @PostConstruct
    public void init() {
        broadcaster = sse.newBroadcaster();

        // Register lifecycle hooks:
        broadcaster.onClose(sink -> {
            System.out.println("[SSE] Client disconnected — sink evicted");
        });
        broadcaster.onError((sink, throwable) -> {
            System.err.println("[SSE] Error on sink: " + throwable.getMessage());
        });
    }

    // Endpoint: clients call GET to subscribe:
    @GET
    @Path("/stream")
    @Produces(MediaType.SERVER_SENT_EVENTS)
    public void subscribe(@Context SseEventSink sink) {
        broadcaster.register(sink);        // add to broadcast list
        System.out.println("[SSE] New client registered. Total: " + broadcaster.size());
    }

    // Endpoint: POST triggers broadcast to ALL connected clients:
    @POST
    @Path("/alerts")
    @Consumes(MediaType.APPLICATION_JSON)
    public Response broadcastAlert(ClusterAlert alert) {
        OutboundSseEvent event = sse.newEventBuilder()
            .name("cluster-alert")
            .data(ClusterAlert.class, alert)
            .build();

        broadcaster.broadcast(event);   // fan-out to ALL registered sinks
        return Response.accepted().build();
    }

    @PreDestroy
    public void cleanup() {
        broadcaster.close();   // gracefully close all connections on shutdown
    }
}
```

---

## :material-broadcast: Unicast — Per-Client Progress Streaming

For streaming progress updates to a **single specific client**:

```java
@GET
@Path("/jobs/{jobId}/progress")
@Produces(MediaType.SERVER_SENT_EVENTS)
public void streamJobProgress(@PathParam("jobId") String jobId,
                               @Context SseEventSink sink,
                               @Context Sse sse) {
    List<String> stages = List.of(
        "Extracting data...",
        "Validating schema...",
        "Transforming records...",
        "Loading to warehouse...",
        "Indexing complete"
    );

    // Run on background thread — never block HTTP thread:
    executor.submit(() -> {
        try {
            for (int i = 0; i < stages.size(); i++) {
                if (sink.isClosed()) break;   // client disconnected — stop!

                OutboundSseEvent event = sse.newEventBuilder()
                    .name("progress")
                    .id("stage-" + (i + 1))
                    .data("stage " + (i + 1) + "/" + stages.size() + ": " + stages.get(i))
                    .build();

                sink.send(event);
                Thread.sleep(1000);    // 1 second between stages
            }

            // Send final completion event:
            sink.send(sse.newEventBuilder().name("complete").data("Job finished!").build());
            sink.close();   // signal server-side EOF to client

        } catch (Exception e) {
            sink.close();
        }
    });
}
```

---

## :material-web: JavaScript Client — `EventSource`

```javascript
// Connect to SSE endpoint:
const source = new EventSource('/api/cluster/stream');

// Listen to named events:
source.addEventListener('cluster-alert', (event) => {
    const alert = JSON.parse(event.data);
    console.log('Alert received:', alert);
    updateDashboard(alert);
});

source.addEventListener('progress', (event) => {
    document.getElementById('progress').innerText = event.data;
});

source.addEventListener('complete', () => {
    console.log('Stream complete!');
    source.close();   // close from client side
});

// Handle connection errors:
source.onerror = (event) => {
    if (event.target.readyState === EventSource.CLOSED) {
        console.log('Connection lost — browser will auto-retry');
    }
};
```

---

## :material-alert: Disconnect Detection — `sink.isClosed()`

```java
// ALWAYS check before writing to prevent exceptions from dead connections:
if (!sink.isClosed()) {
    sink.send(event);
} else {
    // Client disconnected — stop background work immediately:
    executor.shutdownNow();
    break;
}
```

!!! warning "Orphaned background threads"
    If a background task keeps running after the client disconnects (not checking `sink.isClosed()`), the thread wastes resources until it finishes naturally. Always poll `sink.isClosed()` in streaming loops and break on `true`.

---

## :material-key: Key Takeaways — Ch10 SSE

1. **`@Produces(MediaType.SERVER_SENT_EVENTS)` + `void` return** — the two requirements for SSE methods
2. **`Sse` factory** — creates both `OutboundSseEvent.Builder` and `SseBroadcaster`; always `@Context`-injected
3. **`SseBroadcaster.register(sink)`** — subscribe a client; `broadcast(event)` pushes to all registered sinks simultaneously
4. **`onClose` / `onError` hooks** — container-called lifecycle listeners; use to clean up evicted sinks
5. **`sink.isClosed()`** — poll this in every background streaming loop to detect client disconnect early
6. **SSE is unidirectional** — if bidirectional is needed, use Jakarta WebSocket instead

---

[:octicons-arrow-left-24: Back to Day 17 Index](index.md)
