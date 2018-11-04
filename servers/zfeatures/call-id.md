---
title: CallId
caption: CallId
category: servers
permalink: /servers/features/call-id.html
feature:
  artifact: io.ktor:ktor-server-core:$ktor_version
  class: io.ktor.features.CallId
---

The CallId feature allows to identify a request/call and can work along the [CallLogging](/servers/features/call-logging.html) feature.

It has been introduced in Ktor 0.9.5.

### Generating Call IDs 

```kotlin
install(CallId) {
    // Tries to retrieve a callId from an ApplicationCall. You can add several retrievers and all will be executed coalescing until one of them is not null.  
    retrieve { // call: ApplicationCall ->
        call.request.header(HttpHeaders.XRequestId)
    }
    
    // If can't retrieve a callId from the ApplicationCall, it will try the generate blocks coalescing until one of them is not null.
    val counter = atomic(0)
    generate { "generated-call-id-${counter.getAndIncrement()}" }
    
    // Once a callId is generated, this optional function is called to verify if the retrieved or generated callId String is valid. 
    verify { callId: String ->
        callId.isNotEmpty()
    }
    
    // Allows to process the call to modify headers or generate a request from the callId
    reply { call: ApplicationCall, callId: String ->
    
    }

    // Retrieve the callId from a headerName
    retrieveFromHeader(headerName: String)
    
    // Automatically updates the response with the callId in the specified headerName
    replyToHeader(headerName: String)
    
    // Combines both: retrieveFromHeader and replyToHeader in one single call
    header(headerName: String)
}
```

### Extending [CallLogging](/servers/features/call-logging.html)
{: #call-logging-interop }

The CallId feature includes a `callIdMdc` extension method to be used when configuring the CallLogging.
It allows to associate the `callId` to the specified key to be put in the MDC context. 

```kotlin
install(CallLogging) {
    callIdMdc("mdc-call-id")
}
```
