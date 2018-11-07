---
title: HttpRedirect
category: clients
caption: HttpRedirect 
feature:
  artifact: io.ktor:ktor-client-core:$ktor_version
  class: io.ktor.client.features.HttpRedirect
---

By default, the Ktor HTTP Client doesn't follow redirections (except for Apache).
This feature allows you to follow `Location` redirects in a way that works with any HTTP engine.
Its usage is pretty straightforward, and the only configurable property is the `maxJumps` (20 by default) that limits how many redirects are tried before giving up (to prevent infinite redirects).

{% include feature.html %}

## Installation

This feature is installed by default.

## Prevent Installation

```kotlin
val client = HttpClient(HttpClientEngine) {
    followRedirects = false
}
``` 

This feature is included in the `HttpClient` core, so it is always available with the client.
{: .note}
