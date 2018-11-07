---
title: Storages
caption: Session Storages
category: servers
redirect_from:
- /features/sessions/storages.html
---

There are two predefined storages: `SessionStorageMemory`, `DirectoryStorage`. And another composable storage: `CacheStorage`.

`DirectoryStorage` and `CacheStorage` are dependant on the `io.ktor:ktor-server-sessions:$ktor_version` artifact.
{: .note.artifact } 

In this mode, you are just sending a Session Id instead of the actual session contents.
This id is used to store its contents on the server side using a specific `SessionStorage`.
This mode is used when you specify a second argument with a storage in the `cookie` or `header` methods.

Example:

```kotlin
install(Sessions) {
    cookie<SampleSession>("SESSION_FEATURE_SESSION_ID", SessionStorageMemory()) {
        cookie.path = "/"
    }
}
```


## Custom SessionStorage

`SessionStorage` is in charge of storing and retrieving session payload. The interface is *suspendable*,
so you can, and should if it is possible, transfer the data asynchronously.

The data is transferred as a stream and the callee will pass consumers and providers offering the binary payload,
and the callee will be in charge of opening and closing those byte channels.

```kotlin
interface SessionStorage {
    suspend fun write(id: String, provider: suspend (ByteWriteChannel) -> Unit)
    suspend fun invalidate(id: String)
    suspend fun <R> read(id: String, consumer: suspend (ByteReadChannel) -> R): R
}
```

If the storage doesn't provide a meaningful way to store information as a stream, you might want to use
a simplified adaptor that just reads and writes it using `ByteArray`. It can also be used as an example to know
how to deal with the API in its primitive stream-based version.

```kotlin
abstract class SimplifiedSessionStorage : SessionStorage {
    abstract suspend fun read(id: String): ByteArray?
    abstract suspend fun write(id: String, data: ByteArray?): Unit

    override suspend fun invalidate(id: String) {
        write(id, null)
    }

    override suspend fun <R> read(id: String, consumer: suspend (ByteReadChannel) -> R): R {
        val data = read(id) ?: throw NoSuchElementException("Session $id not found")
        return consumer(ByteReadChannel(data))
    }

    override suspend fun write(id: String, provider: suspend (ByteWriteChannel) -> Unit) {
        return provider(CoroutineScope(Dispatchers.IO).reader(coroutineContext, autoFlush = true) {
            val data = ByteArrayOutputStream()
            val temp = ByteArray(1024)
            while (!channel.isClosedForRead) {
                val read = channel.readAvailable(temp)
                if (read <= 0) break
                data.write(temp, 0, read)
            }
            write(id, data.toByteArray())
        }.channel)
    }
}
```