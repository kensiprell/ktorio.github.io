---
title: Chat
caption: "Guides: How to implement a chat with WebSockets"
category: quickstart
---

In this tutorial you will learn to make a chat application using Ktor.
We are going to use WebSockets for real-time, bidirectional communication.

This is an advanced tutorial, and you must understand some basic concepts about Ktor, so if you haven't done so already, please see the [guide on creating a website](/quickstart/guides/website.html).

To achieve this we are going to use the [Routing], [WebSockets], and [Sessions] features.

[Routing]: /servers/features/routing.html
[WebSockets]: /servers/features/websockets.html
[Sessions]: /servers/features/sessions.html

**Table of contents:**

* TOC
{:toc}

## Setting Up the Project

The first step is to set up a project.
You can follow the [Quick Start](/quickstart/index.html) guide or use the following tool to create one:

{% include preconfigured-form.html hash="dependency=ktor-sessions&dependency=routing&dependency=ktor-websockets&artifact-name=chat" %}

## Understanding WebSockets

WebSockets is a subprotocol of HTTP.
It starts as a normal HTTP request with an upgrade request header, and the connection switches to bidirectional communication instead of continuing with a request/response model.

The smallest unit of transmission that can be sent as part of the WebSocket protocol is a `Frame`.
For a single message, TCP can be fragmented into several packets.
A WebSocket Frame defines a type and a length, and thus could be transmitted in several TCP packets but, it will be reassembled into a single Frame.

You can imagine Frames as WebSocket messages.
Frames can be the following types: text, binary, close, ping, and pong.

You will normally handle `Text` and `Binary` frames, and the other types will be handled by Ktor in most cases (though you can use raw mode).

Here you can read more about the [WebSockets feature](/servers/features/websockets.html).  

## WebSocket Route

The first step is to create a route for the WebSocket, and we are going to define the `/chat` route.
We are going to start with an echo WebSocket route that will "echo" the same message that you sent right back to you.

`WebSocket` routes are intended to be long-lived.
Since it is a suspend block and uses lightweight Kotlin Coroutines, you can handle hundreds of thousands of connections at once (depending on the machine and the complexity) while keeping your code easy to read and write.

```kotlin
routing {
    webSocket("/chat") { // this: DefaultWebSocketSession
        while (true) {
            val frame = incoming.receive()
            when (frame) {
                is Frame.Text -> {
                    val text = frame.readText()
                    outgoing.send(Frame.Text(text))
                }
            }
        }
    }
}
```

## Keeping a Set of Opened Connections

We can use a Set to keep a list of opened connections.
We can use a plain `try...finally` to keep track of them.
Since Ktor is multithreaded by default, we should use thread-safe collections or [limit the body to a single thread with newSingleThreadContext](https://github.com/Kotlin/kotlinx.coroutines/blob/master/coroutines-guide.md#coroutine-context-and-dispatchers){:target="_blank"}. 

```kotlin
routing {
    val wsConnections = Collections.synchronizedSet(LinkedHashSet<DefaultWebSocketSession>())
    
    webSocket("/chat") { // this: DefaultWebSocketSession
        wsConnections += this
        try {
            while (true) {
                val frame = incoming.receive()
                // ...
            }
        } finally {
            wsConnections -= this
        }
    }
}
```

## Propagating a Message to All Connections

Now that we have a set of connections, we can iterate over them and use the session to send the frames we need.
Every time a user sends a message, we are going to propagate it to all connected clients.

```kotlin
routing {
    val wsConnections = Collections.synchronizedSet(LinkedHashSet<DefaultWebSocketSession>())
    
    webSocket("/chat") { // this: DefaultWebSocketSession
        wsConnections += this
        try {
            while (true) {
                val frame = incoming.receive()
                when (frame) {
                    is Frame.Text -> {
                        val text = frame.readText()
                        // Iterate over all the connections
                        for (conn in wsConnections) {
                            conn.outgoing.send(Frame.Text(text))
                        }
                    }
                }
            }
        } finally {
            wsConnections -= this
        }
    }
}
```

## Assigning Names to Users/Connections

We might want to associate some information, like a name to an opened connection.
We can create an object that includes the WebSocketSession and store it instead ike this:

```kotlin
class ChatClient(val session: DefaultWebSocketSession) {
    companion object { var lastId = AtomicInteger(0) }
    val id = lastId.getAndIncrement()
    val name = "user$id"
}

routing {
    val clients = Collections.synchronizedSet(LinkedHashSet<ChatClient>())
    
    webSocket("/chat") { // this: DefaultWebSocketSession
        val client = ChatClient(this)
        clients += client
        try {
            while (true) {
                val frame = incoming.receive()
                when (frame) {
                    is Frame.Text -> {
                        val text = frame.readText()
                        // Iterate over all the connections
                        val textToSend = "${client.name} said: $text"
                        for (other in clients.toList()) {
                            other.session.outgoing.send(Frame.Text(textToSend))
                        }
                    }
                }
            }
        } finally {
            clients -= client
        }
    }
}
```

## Exercises

### Create a Client

Create a JavaScript client that connects to this endpoint and serve it with Ktor.

### JSON

Use [kotlinx.serialization](https://github.com/Kotlin/kotlinx.serialization) to send and receive value objects.

