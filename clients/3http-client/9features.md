---
title: Features
caption: Http Client Features
category: clients
permalink: /clients/http-client/features.html
children: /clients/http-client/features/
---

Similar to the server, Ktor supports features on the client. And it has the same design:
there is a pipeline for client HTTP requests, and there are interceptors and installable features.

{% include children_list.html context=page.children %}

### Creating Custom Features

If you want to create features, you can use the [standard features](https://github.com/ktorio/ktor/tree/master/ktor-client/ktor-client-core/src/io/ktor/client/features) as a reference.

You can also check the [HttpRequestPipeline.Phases](https://github.com/ktorio/ktor/blob/master/ktor-client/ktor-client-core/src/io/ktor/client/request/HttpRequestPipeline.kt)
and [HttpResponsePipeline.Phases](https://github.com/ktorio/ktor/blob/master/ktor-client/ktor-client-core/src/io/ktor/client/response/HttpResponsePipeline.kt)
to understand the interception points available.
{: .note.tip}
