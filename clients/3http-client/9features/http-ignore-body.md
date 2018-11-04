---
title: HttpIgnoreBody
category: clients
caption: HttpIgnoreBody 
permalink: /clients/http-client/features/http-ignore-body.html
feature:
  artifact: io.ktor:ktor-client-core:$ktor_version
  class: io.ktor.client.features.HttpIgnoreBody
---

This feature discards the body of the response.

{% include feature.html %}

## Install

```kotlin
val client = HttpClient(HttpClientEngine) {
    install(HttpIgnoreBody)
}
```

Use this if you are only interested in the response headers, and you cannot use the HEAD verb.
This will use less memory and will execute faster.
{: .note}
