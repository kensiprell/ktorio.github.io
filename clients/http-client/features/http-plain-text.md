---
title: HttpPlainText
category: clients
caption: HttpPlainText
feature:
  artifact: io.ktor:ktor-client-core:$ktor_version
  class: io.ktor.client.features.HttpPlainText
---

This feature processes the request content as plain text using a specific charset defined by `defaultCharset`. 
It will also process the response content as plain text.

{% include feature.html %}

## Installation

```kotlin
val client = HttpClient(HttpClientEngine) {
    install(HttpPlainText) {
        defaultCharset = Charsets.UTF_8
    }
}
```

Bear in mind that the default charset is the JVM's charset, which could be different between systems. 
That's why it is recommended to specify the default charset.
{: .note}
