---
title: Examples
category: clients
permalink: /clients/http-client/examples.html
caption: HTTP Client Examples 
---

### Interchanging JSON: Ktor server / Ktor client
{: #example-json }

```kotlin
fun main(args: Array<String>) {
    val server = embeddedServer(
        Netty,
        port = 8080,
        module = Application::mymodule
    ).apply {
        start(wait = false)
    }

    runBlocking {
        val client = HttpClient(Apache) {
            install(JsonFeature) {
                serializer = GsonSerializer {
                    // .GsonBuilder
                    serializeNulls()
                    disableHtmlEscaping()
                }
            }
        }

        val message = client.post<HelloWorld> {
            url(URL("http://127.0.0.1:8080/"))
            contentType(ContentType.Application.Json)
            body = HelloWorld(hello = "world")
        }

        println("CLIENT: Message from the server: $message")

        client.close()
        server.stop(1L, 1L, TimeUnit.SECONDS)
    }
}

data class HelloWorld(val hello: String)

fun Application.mymodule() {
    install(ContentNegotiation) {
        gson {
            setPrettyPrinting()
        }
    }
    routing {
        post("/") {
            val message = call.receive<HelloWorld>()
            println("SERVER: Message from the client: $message")
            call.respond(HelloWorld(hello = "response"))
        }
    }
}
```

You can check the [ktor-samples](https://github.com/ktorio/ktor-samples) and [ktor-exercises](https://github.com/ktorio/ktor-exercises) repositories for samples and exercises.
{: .note }
