---
title: Requests
caption: HTTP Client Requests
category: clients
permalink: /clients/http-client/calls/requests.html
---

## Simple Requests

The basic usage is *super* simple: you just have to instantiate an `HttpClient` instance,
specify an engine, for example,
[`Apache`](/clients/http-client/engines.html#apache),
[`OkHttp`](/clients/http-client/engines.html#okhttp),
[`Android`](/clients/http-client/engines.html#android),
[`Ios`](/clients/http-client/engines.html#ios),
[`Jetty`](/clients/http-client/engines.html#jetty)
or [`CIO`](/clients/http-client/engines.html#cio),
and start making requests using one of the many convenience methods available.

Since Ktor 0.9.3 you can omit the engine, and Ktor will choose an engine among the ones that are available from the included artifacts using a ServiceLoader. 

First you need to instantiate the client:   

```kotlin
val client = HttpClient(Apache)
```

Then, to perform a `GET` request and fully read a `String`:

```kotlin
val htmlContent = client.get<String>("https://en.wikipedia.org/wiki/Main_Page")
```

And in the case you are interested in the raw bits, you can read a `ByteArray`:

```kotlin
val bytes: ByteArray = client.call("http://127.0.0.1:8080/").response.readBytes()
```

It is possible to customize the request and to stream the request and response payloads, but you can also just call a convenience extension method like `HttpClient.get` to do a `GET` request to receive the specified type directly (for example `String`).
{: .note}

After you finish working with the client, it should be closed in order to properly stop the underlying engine.

```kotlin
client.close()
```

If you want to use a client to make only one request, consider `use`ing it. The client will be automatically closed once the passed block has been executed.

```kotlin
val bytes = HttpClient(Apache).use { client ->
    client.call("http://127.0.0.1:8080/").response.readBytes()
}
```

## Custom Requests

We cannot rely only on *get* requests, and Ktor allows you to build complex
requests with any of the HTTP verbs with the flexibility to process responses in many ways.

### The `call` Method
{: #call-method }

The HttpClient `call` method returns an `HttpClientCall` and allows you to perform
simple untyped requests.

You can read the content using `response: HttpResponse`. For further information, check out the [receiving content using HttpResponse](/clients/http-client/calls/responses.html) section. 

```kotlin
val call = client.call("http://127.0.0.1:8080/") {
    method = HttpMethod.Get
}
println(call.response.readText())
```

### The `request` Method
{: #request-method }

In addition to `call`, there is a `request` method for performing a typed request,
[receiving a specific type](/clients/http-client/calls/responses.html#receive) like String, HttpResponse, or an arbitrary class. You have to specify the URL and the method when building the request. 

```kotlin
val call = client.request<String> {
    url(URL("http://127.0.0.1:8080/"))
    method = HttpMethod.Get
}
```

### The `get`, `post`, `put`, and `delete` Methods
{: #shortcut-methods }

Similar to `request`, there are several extension methods to perform requests with the most common HTTP verbs: `GET`, `POST`, `PUT` and `DELETE`.

`PATCH`, `HEAD` and `OPTIONS` have special HTTP semantics, and they do not have shortcut methods although you can use the [`request`](#request-method) and [`call`](#call-method) methods to perform requests with those verbs. 
{: .note }

```kotlin
val text = client.post<String>("http://127.0.0.1:8080/")
```

When calling request methods, you can provide a lambda to build the request parameters like the URL, the HTTP method (verb), the body, and the headers. The `HttpRequestBuilder` looks like this:

```kotlin
class HttpRequestBuilder : HttpMessageBuilder {
    val url: URLBuilder
    var method: HttpMethod
    val headers: HeadersBuilder
    var body: Any = EmptyContent
    val executionContext: CompletableDeferred<Unit>
    fun header(key: String, value: String)
    fun headers(block: HeadersBuilder.() -> Unit)
    fun url(block: URLBuilder.(URLBuilder) -> Unit)
}
```

The `HttpClient` class only offers some basic functionality, and all the methods for building requests are exposed as extensions. You can check out the standard available [HttpClient build extension methods](https://github.com/ktorio/ktor/blob/master/ktor-client/ktor-client-core/src/io/ktor/client/request/builders.kt).
{: .note.api}

### The `submitForm` and `submitFormWithBinaryData` Methods
{: #submit-form }

There are a couple of convenience extension methods for submitting form information.

The `submitForm` method:

`submitForm(formData: Parameters = Parameters.Empty, encodeInQuery: Boolean = false, block: HttpRequestBuilder.() -> Unit = {})`

It allows you to make a request with the `Parameters` encoded in the query string (`GET` by default) or to make a request with the `Parameters` encoded as multipart (`POST` by default), depending on the `encodeInQuery` parameter.

The `submitFormWithBinaryData` method:

`submitFormWithBinaryData(formData: List<PartData>, block: HttpRequestBuilder.() -> Unit = {}): T`

It allows you to generate a multipart POST request from a list of `PartData`.
`PartData` can be `PartData.FormItem`, `PartData.BinaryItem`, or `PartData.FileItem`.

You can use the `formData` builder to build a list of `PartData`:

```kotlin
val data: List<PartData> = formData {
    // Can append: String, Number, ByteArray and Input.
    append("hello", "world")
    append("number", 10)
    append("ba", byteArrayOf(1, 2, 3, 4))
    append("input", inputStream.asInput())
    // Allow to set headers to the part:
    append("hello", "world", headersOf("X-My-Header" to "MyValue"))
}
```

### Specifying Custom Headers
{: #custom-headers}

When building requests with `HttpRequestBuilder`, you can set custom headers.
There is a final property `val headers: HeadersBuilder` that inherits from `StringValuesBuilder`. You can add or remove headers using it or with the `header` convenience methods.

```kotlin
// this : HttpMessageBuilder

// Convenience method to add a header
header("My-Custom-Header", "HeaderValue")

// Calls methods from the headers: HeadersBuilder to manipulate the headers
headers.clear()
headers.append("My-Custom-Header", "HeaderValue")
headers.appendAll("My-Custom-Header", listOf("HeaderValue1", "HeaderValue2"))
headers.remove("My-Custom-Header")

// Applies the headers with the `headers` convenience method
headers { // this: HeadersBuilder
    clear()
    append("My-Custom-Header", "HeaderValue")
    appendAll("My-Custom-Header", listOf("HeaderValue1", "HeaderValue2"))
    remove("My-Custom-Header")
}
``` 


## Specifying a Body for Requests

For `POST` and `PUT` requests, you can set the `body` property:

```kotlin
client.post<Unit> {
    url(URL("http://127.0.0.1:8080/"))
    body = // ...
}
```

The `HttpRequestBuilder.body` property can be a subtype of `OutgoingContent` as well as a `String` instance:

* `body = "HELLO WORLD!"`
* `body = TextContent("HELLO WORLD!", ContentType.Text.Plain)`
* `body = ByteArrayContent("HELLO WORLD!".toByteArray(Charsets.UTF_8))`
* `body = LocalFileContent(File("build.gradle"))`
* `body = JarFileContent(File("myjar.jar"), "test.txt", ContentType.fromFileExtension("txt").first())`
* `body = URIFileContent(URL("https://en.wikipedia.org/wiki/Main_Page"))`

If you install the *JsonFeature* and set the content type to `application/json`,
you can use arbitrary instances as the `body`, and they will be serialized as JSON:

```kotlin
data class HelloWorld(val hello: String)

val client = HttpClient(Apache) {
    install(JsonFeature) {
        serializer = GsonSerializer {
            // Configurable .GsonBuilder
            serializeNulls()
            disableHtmlEscaping()
        }
    }
}

client.post<Unit> {
    url(URL("http://127.0.0.1:8080/"))
    body = HelloWorld(hello = "world")
}
```

Alternatively (using the integrated `JsonSerializer`):

```kotlin
val json = io.ktor.client.features.json.defaultSerializer()
client.post<Unit>() {
    url(URL("http://127.0.0.1:8080/"))
    body = json.write(HelloWorld(hello = "world")) // Generates an OutgoingContent
}
```

Or using Jackson (JVM only):

```kotlin
val json = jacksonObjectMapper()
client.post<Unit> {
    url(URL("http://127.0.0.1:8080/"))
    body = TextContent(json.writeValueAsString(userData), contentType = ContentType.Application.Json)
}
```

Remember that your classes must be *top-level* to be recognized by `Gson`. If you try to send a class that is inside a function, the feature will send a *null*.
{: .note}

## Uploading multipart/form-data
{: #multipart-form-data }

Starting with 0.9.4 Ktor HTTP Client has support for making MultiPart requests.
The idea is to use the `MultiPartFormDataContent(parts: List<PartData>)` as `OutgoingContent` for the body of the request.

The easiest way is to use the [`submitFormWithBinaryData` method](#submit-form).

Alternatively you can set the body directly:

```kotlin
val request = client.request {
    method = HttpMethod.Post
    body = FormDataContent(formData {
        append("key", "value")
    })
}
```
