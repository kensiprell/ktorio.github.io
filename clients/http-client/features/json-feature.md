---
title: JsonFeature
category: clients
caption: JsonFeature
feature:
  artifact: io.ktor:ktor-client-json:$ktor_version
  class: io.ktor.client.features.json.JsonFeature
---

This feature processes the request and the response payloads as JSON, serializing and de-serializing them using a specific `serializer: JsonSerializer`.

```kotlin
val client = HttpClient(HttpClientEngine) {
    install(JsonFeature)
}
```

See the [complete example using JSON](/clients/http-client/examples.html#example-json).

{% include feature.html %}

## Serializers

The `JsonFeature` has a default serializer based on a ServiceLoader on the JVM
and a serializer based on [kotlinx.serialization](/kotlinx/serialization.html) for Native.

You can also get the default serializer by calling `io.ktor.client.features.json.defaultSerializer()`

### Gson
{: #gson }

```kotlin
val client = HttpClient(HttpClientEngine) {
    install(JsonFeature) {
        serializer = GsonSerializer()
    }
}
```

To use this feature, you need to include the `io.ktor:ktor-client-gson` artifact.
{: .note.artifact }

### Jackson
{: #jackson }

```kotlin
val client = HttpClient(HttpClientEngine) {
    install(JsonFeature) {
        serializer = JacksonSerializer()
    }
}
```

To use this feature, you need to include the `io.ktor:ktor-client-jackson` artifact.
{: .note.artifact }

### Kotlinx.Serialization
{: #kotlinx-serialization }

```kotlin
val client = HttpClient(HttpClientEngine) {
    install(JsonFeature) {
        serializer = KotlinxSerializer()
    }
}
```

To use this feature, you need to include the `io.ktor:ktor-client-json-jvm` artifact on the JVM.
{: .note.artifact }
