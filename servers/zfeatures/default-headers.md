---
title: Default Headers
caption: Send Headers Automatically
category: servers
permalink: /servers/features/default-headers.html
feature:
  artifact: io.ktor:ktor-server-core:$ktor_version
  class: io.ktor.features.DefaultHeaders
redirect_from:
- /features/default-headers.html
---

This feature adds a default set of headers to HTTP responses. The list of headers is customizable, and the `Date` header is cached
to avoid building complex strings on each response.   

{% include feature.html %}

### Usage

```kotlin
fun Application.main() {
  ...
  install(DefaultHeaders)
  ...
}
```

This will add `Date` and `Server` headers to each HTTP response.

### Configuration
 
* `header(name, value)` will add another header to the list of default headers

```kotlin
fun Application.main() {
  ...
  install(DefaultHeaders) {
    header("X-Developer", "John Doe") // will send this header with each response
  }
  ...
}
```

* default `Server` header can be overriden by specifying your custom header:

```kotlin
fun Application.main() {
  ...
  install(DefaultHeaders) {
    header(HttpHeaders.Server, "Konstructor") 
  }
  ...
}
```

* The default `Date` header cannot be overridden. If you need to override it, do not install the `DefaultHeaders` feature and instead 
intercept the call manually 