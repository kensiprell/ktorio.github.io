---
title: Gson
caption: JSON support using Gson
category: servers
feature:
  artifact: io.ktor:ktor-gson:$ktor_version
  class: io.ktor.gson.GsonConverter
redirect_from:
- /features/gson.html
- /features/content-negotiation/gson.html
---

The GSON feature allows you to handle JSON content in your application easily using
the [google-gson](https://github.com/google/gson) library.

This feature is a [ContentNegotiation](/servers/features/content-negotiation.html) converter.

{% include feature.html %}

## Basic usage

Install the feature by registering a JSON content converter using Gson:

```kotlin
install(ContentNegotiation) {
    gson {
        // Configure Gson here
    }
}
```

The `gson` block is a convenient method for:

```kotlin
register(ContentType.Application.Json, GsonConverter(GsonBuilder().apply {
    // ...
}.create()))
```

## Configuration

Inside the `gson` block, you have access to the [GsonBuilder](https://google.github.io/gson/apidocs/com/google/gson/GsonBuilder.html)
used to install the ContentNegotiation. To give you an idea of what is available:

```kotlin
install(ContentNegotiation) {
    gson {
        setPrettyPrinting()
        
        disableHtmlEscaping()
        disableInnerClassSerialization()
        enableComplexMapKeySerialization()

        serializeNulls()

        serializeSpecialFloatingPointValues()
        excludeFieldsWithoutExposeAnnotation()
        
        setDateFormat(...)

        generateNonExecutableJson()

        setFieldNamingPolicy()
        setLenient()
        setLongSerializationPolicy(...)
        setExclusionStrategies(...)
        setVersion(0.0)
        addDeserializationExclusionStrategy(...)
        addSerializationExclusionStrategy(...)
        excludeFieldsWithModifiers(Modifier.TRANSIENT)
        
        registerTypeAdapter(...)
        registerTypeAdapterFactory(...)
        registerTypeHierarchyAdapter(..., ...)
    }
}
```