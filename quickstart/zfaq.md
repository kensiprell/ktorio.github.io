---
title: FAQ
caption: Frequently Asked Questions
category: quickstart
permalink: /quickstart/faq.html
priority: 100
redirect_from:
  - /servers/faq.html
---

In this section we provide answers to the questions you frequently ask us.

Can't find an answer for your question? Head over to our #ktor [Kotlin Slack](http://slack.kotlinlang.org/){: target="_blank" rel="noopener"} channel, and we will try to help you!
{: .note}

**Table of contents:**

* TOC
{:toc}

## What is the proper way to pronounce ktor?
{: #pronounce }

`kay-tor`

## How do I ask questions, report bugs, contact you, contribute, give feedback, etc.?
{: #feedback }

Depending on the content, you might consider several channels:

* **GitHub:** feature requests, suggestions, proposals, bugs, and PRs
* **Slack:** questions, troubleshooting, guidance, etc.
* **StackOverflow:** questions

**Rationale:**

For questions or troubleshooting we highly recommend you use Slack or StackOverflow.

Imagine using GitHub issues for questions and troubleshooting. Everyone subscribed to a repository would be notified of each and every comment. Troubleshooting usually requires several questions and answers, and that would result in a lot of emails just for a single question. The people who only want to keep up with bugs, fixes, new or proposed features, etc. would be overwhelmed.

If you have enough time or you prefer not to join Slack, you can also ask questions at StackOverflow. However, with Slack being a hybrid between chat and forum, we can contact each other faster and troubleshoot things in less time.

When troubleshooting, if you determine that there is a bug or something to improve, you can report it at GitHub. Since a bug (once confirmed) reported only on Slack could be forgotten or overlooked, please use GitHub for confirmed issues.

**Pull Requests:**

If you have a bugfix or functionality you think would be worth including in Ktor, you can create a PR.

Keep in mind that we usually review and merge PRs in batches, so the PR could be outstanding for a few weeks. We encourage you to contribute if you can!

If you have a bugfix that you need to use right away, we recommend you fork Ktor, apply your fix, compile it yourself, and temporarily publish a patched version in your own artifactory, bintray or similar and use that version until it is merged and a new version is released or prereleased (since the timing might not be aligned with your needs).

## What does CIO mean?
{: #cio }

CIO stands for Coroutine-based I/O. Usually we call it an engine that uses Kotlin and Coroutines to implement the logic implementing an IETF RFC or another protocol without relying on external JVM libraries.

## Ktor imports are not being resolved. Imports are red.
{: #ktor-artifact }

Ensure that you are including the ktor artifact. For example, for Gradle and the Netty engine it would be:
```kotlin
dependencies {
  compile("io.ktor:ktor-server-netty:$ktor_version")
}
```
* For gradle, check: <http://ktor.io/quickstart/gradle.html#engine>
* For maven, check: <http://ktor.io/quickstart/maven.html>

## Does ktor provide a way to catch IPC signals (e.g. SIGTERM or SIGINT) so the server shutdown can be handled gracefully?
{: #sigterm }

If you are running a `DevelopmentEngine` or `EngineMain`, it will be handled automatically.

Otherwise, you will have to [handle it manually](https://github.com/ktorio/ktor/blob/6c724f804bd6f25158d284d05c49235c67573019/ktor-server/ktor-server-cio/src/io/ktor/server/cio/EngineMain.kt#L18).
You can use the JVM's `Runtime.getRuntime().addShutdownHook` facility.

## How do I get the client IP behind a proxy?
{: #proxy-ip }

The property `call.request.origin` gives connection information about the original caller (the proxy) if the proxy provides proper headers and the feature `XForwardedHeaderSupport` is installed.

## I get the error 'java.lang.IllegalStateException: No instance for key AttributeKey: Locations'
{: #no-attribute-key-locations }

You get this error if you try to use the locations feature without actually installing it. Check the locations feature: <https://ktor.io/features/locations.html>

## I get a 406 error with a client not sending an Accept header. With WRK I'm getting Non-2xx responses after adding JSON support
{: #missing-accept-issue }

There is a [known issue](https://github.com/ktorio/ktor/issues/38) in Ktor <= 0.9.1, that when a client does not send an Accept header, the content negotiation assumes that nothing should be accepted. Since 0.9.2-alpha-1, Ktor assumes it should accept everything when no Accept header is sent.

## How can I test the latest commits on master?
{: #bleeding-edge }

You can use jitpack to get builds from master that are not yet released: <https://jitpack.io/#ktorio/ktor>
Also you can [build Ktor from source](/advanced/building-from-source.html) and use your mavenLocal repository for the artifact.

## How can I be sure of which version of Ktor am I using?
{: #ktor-version-used }

You can use the [`DefaultHeaders` feature](/servers/features/default-headers.html) that will send a server header with the Ktor version in it. Something like this should be sent as part of the response headers: `Server: ktor-server-core/0.9.2-alpha-3 ktor-server-core/0.9.2-alpha-3`

## Website accessibility tips and tricks
{: #website-tricks }

You can use the keys <kbd>s</kbd> (search), <kbd>t</kbd> (Github file finder flavor) or <kbd>#</kbd> to access the search function on any page of the documentation website. The <kbd>#</kbd> version will limit the search to the sections in the current page.

In the search function you can either select the options with your mouse or fingers or use the keyboard arrows <kbd>↑</kbd> <kbd>↓</kbd> and the return key <kbd>⏎</kbd> to go to the currently selected page.

The search function only uses page titles and keywords for the search. It is also possible to do a Google search on the `ktor.io` domain to do a full text search on all of its contents.

Long code fragments that are folded can be expanded by either clicking on the `'+'`/`'-'` symbol that always appears in the top left corner of mobile devices or on hover on devices with a mouse. You can also double click the fragment to expand it. In addition to expanding it, this action selects the text so you can easily copy the fragments with <kbd>cmd</kbd> + <kbd>c</kbd> on mac, or <kbd>ctrl</kbd> + <kbd>c</kbd> on other operating systems.

You can click on the headings and notes to get an anchored link to the sections. After clicking, you can copy the new URL in your browser, including the `#` to link to a specific section.

## My route is not being executed, how can I debug it?
{: #route-not-executing }

Ktor provides a tracing mechanism for the routing feature to help troubleshooting routing decisions. Check the [Tracing the routing decisions](/servers/features/routing.html#tracing) section on the Routing page.

## I get a `io.ktor.pipeline.InvalidPhaseException: Phase Phase('YourPhase') was not registered for this pipeline`.
{: #invalid-phase }

This means that you are trying to use a phase that is not registered as a reference for another phase. This might happen for example in the Routing feature if you try to register a phase relation inside a node, but the phase referenced is defined in another ancestor Route node. Since route phases and interceptors are later merged, it should work, but you need to register it in your Route node:

```kotlin
route.addPhase(PhaseDefinedInAncestor)
route.insertPhaseAfter(PhaseDefinedInAncestor, MyNodePhase)
```

## I get a `io.ktor.server.engine.BaseApplicationResponse$ResponseAlreadySentException: Response has already been sent`
{: #response-already-sent }

This means that you or a feature or interceptor have already called `call.respond*` functions and you are calling it
again.

## How can I subscribe to Ktor events?
{: #ktor-events }

There is a page [explaining the Ktor's application-level event system](/advanced/events.html).

## I get an `Exception in thread "main" com.typesafe.config.ConfigException$Missing: No configuration setting found for key 'ktor'` exception
{: #cannot-find-application-conf }

This means that Ktor was not able to find the `application.conf` file. Re-check that it is in the `resources` folder and that the resources folder is marked as such. You should consider setting up a project using the [project generator](/quickstart/generator.html) or the [IntelliJ plugin](/quickstart/quickstart/intellij-idea/plugin.html) to get a working project as a base.

## Can I use Ktor on Android?
{: #android-support }

Ktor 0.9.3 and lower is known to work on Android 7 or greater (API 24). It will fail in lower versions like Android 5.

In unsupported versions it would fail with an exception similar to:

```
E/AndroidRuntime: FATAL EXCEPTION: main Process: com.mypackage.example, PID: 4028 java.lang.NoClassDefFoundError:
io.ktor.application.ApplicationEvents$subscribe$1 at io.ktor.application.ApplicationEvents.subscribe(ApplicationEvents.kt:18) at
io.ktor.server.engine.BaseApplicationEngine.<init>(BaseApplicationEngine.kt:29) at
io.ktor.server.engine.BaseApplicationEngine.<init>(BaseApplicationEngine.kt:15) at
io.ktor.server.netty.NettyApplicationEngine.<init>(NettyApplicationEngine.kt:17) at io.ktor.server.netty.Netty.create(Embedded.kt:10) at
io.ktor.server.netty.Netty.create(Embedded.kt:8) at io.ktor.server.engine.EmbeddedServerKt.embeddedServer(EmbeddedServer.kt:50) at
io.ktor.server.engine.EmbeddedServerKt.embeddedServer(EmbeddedServer.kt:40) at
io.ktor.server.engine.EmbeddedServerKt.embeddedServer$default(EmbeddedServer.kt:27)
```

For more information, check [Issue #495](https://github.com/ktorio/ktor/issues/495) and this [StackOverflow
question](https://stackoverflow.com/questions/49945584/attempting-to-run-an-embedded-ktor-http-server-on-android),

## CURL -I returns a 404 Not Found
{: #curl-head-not-found }

`CURL -I` is an alias of `CURL --head` that performs a `HEAD` request.
By default Ktor doesn't handle `HEAD` requests for `GET` handlers, so you might get something like:

```kotlin
curl -I http://localhost:8080
HTTP/1.1 404 Not Found
Content-Length: 0
```

for:

```kotlin
routing {
  get("/") { call.respondText("HELLO") }
}
```

Ktor can automatically handle `HEAD` requests but requires you to first install the [`AutoHeadResponse` feature](/servers/features/autoheadresponse.html).

## I get an infinite redirect when using the `HttpsRedirect` feature
{: #infinite-redirect }

The most probable cause is that your backend is behind a reverse-proxy or a load balancer, and the reverse-proxy is making normal HTTP requests to your backend. Therefore, the HttpsRedirect feature inside your Ktor backend believes that it is a normal HTTP request and responds with the redirect.

Normally, reverse-proxies send some headers describing the original request (like it was https or the original IP address), and there is the feature [`XForwardedHeaderSupport`](/servers/features/forward-headers.html)
to parse those headers so the [`HttpsRedirect`](/servers/features/https-redirect.html) feature knows that the original request was HTTPS.


## I get a `UnsafeHeaderException: Header Content is controlled by the engine and cannot be set explicitly` exception
{: #UnsafeHeaderException }

Check the [0.9.4 migration guide](/quickstart/migration/0.9.4.html#UnsafeHeaderException) for more information about how to fix it.
