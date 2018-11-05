---
title: ExceptSuccess
category: clients
caption: ExceptSuccess 
feature:
  artifact: io.ktor:ktor-client-core:$ktor_version
  class: io.ktor.client.features.ExpectSuccess
---

This feature checks that the response is OK (code < 300), and if not, it throws a `BadResponseStatus`. 

{% include feature.html %}

## Installation

This feature is installed by default.

## Prevent Installation

You can prevent installation of this feature by setting the `HttpClient.expectSuccess` property to `false`:

```kotlin
val client = HttpClient(HttpClientEngine) {
    expectSuccess = false
}
```
