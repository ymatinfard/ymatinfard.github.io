---
layout: post
title:  "Optimizing Server Driven UI with Multi-Level Caching"
date:   2025-01-30 18:23:00 GMT+1
categories: android ui server-driven
---

![starting-image](/assets/images/post/multi_level_caching.png){: width="512" height="512" }

Server-Driven UI (SDUI) is a way to build Android apps where the server controls how the screen looks. This means we can change the app's screens without making users download a new version. However, there is a problem: it can be slow because the app must wait for data from the server.

## The Challenge: Speed
When a user opens a screen in an SDUI app, these things happen:
1. The app asks the server for UI data
2. The server processes this request
3. The app receives the data and shows it on screen

This process can be slow, and users do not like waiting.

## The Solution: Multi-Level Cache
To make the app faster, we use two types of storage:

### Memory Cache

- Stores UI data in the device's RAM
- Very fast access
- Limited storage space
- Data is lost when app closes

### Disk Cache

- Stores UI data in device storage
- Slower than memory but still fast
- More storage space
- Data stays when app closes

## How It Works
When the app needs to show a screen:
1. First, it looks in Memory Cache
   - If found: Shows screen immediately
   - If not found: Goes to step 2

2. Then, it looks in Disk Cache
   - If found: Copies to Memory Cache and shows screen
   - If not found: Goes to step 3

3. Finally, it asks the Server
   - Gets data from server
   - Saves to both Memory and Disk Cache
   - Shows screen

## Result
  - Better Speed: Shows screens faster
  - Works Offline: Can show screens without internet
  - Less Server Load: Fewer requests to the server
  - Saves Data: Uses less internet data
  - Better Experience: Users see screens quickly

When designing a caching solution, the goal is to keep the logic encapsulated inside a cache class, so the rest of the application interacts with it as if by magic - without worrying about whether data is coming from the cache or being fetched from a remote server. However, to ensure reusability, we need to extract some variable operations, such as:
Fetching fresh data from the server
Storing data in a local cache (e.g., SharedPreferences, Room database)
Storing data in memory for quick access. By keeping these operations external, we make our caching mechanism adaptable to different use cases.

## Here's how we can design it:

### Representing state

```kotlin
data class Data<T>(val content: T? = null, val error: Throwable? = null, val loading: Boolean = false)
```
### Memory and storage
```kotlin
class CacheManager<Param: Any, Domain: Any>(
  private val fetchFromMemory: (Params) -> Domain? = { null }
  private val saveToMemory: (Params, Domain) -> Unit = { _, _ -> }
  private val fetchFromStorage: suspend (Params) -> Domain? = { null }
  private val saveToStorage: suspend (Params, Domain) -> Unit = { _, _ -> }
)

```
### Data access
```kotlin
fun observe(params: Params, forceReload: Boolean): Flow<Data<Domain>> = flow {
    val cachedData = fetchFromMemory(params)
    val shouldLoad = cachedData == null || forceReload
    emitAll(
        fetchFromStorage(params, cachedData)
            .flatMapMerge { storageData ->
                fetchFromNetworkIfNeeded(shouldLoad, storageData, params)
            }
            .onStart {
                emit(Data(content = cachedData, loading = shouldLoad))
            }
            .distinctUntilChanged()
    )
}

```
### Fetch from storage
```kotlin
    private suspend fun fetchFromStorage(
        params: Params,
        cachedData: Domain?
    ): Flow<Data<Domain>> =
        if (cachedData != null) {
            flowOf(Data(cachedData))
        } else {
            flow {
                val storageData = fetchFromStorage(params)
                if (storageData != null) {
                    saveToMemory(params, it)
                }
                emit(Data(content = storageData))
            }.flowOn(ioDispatcher)
                .catch { throwable ->
                    val loadingErrorData = Data<Domain>(error = throwable, loading = true)
                    emit(loadingErrorData)
                }
        }
```
### Fetch from network
```kotlin
  private suspend fun fetchFromNetwork(
        params: Params,
        cachedData: Domain?
    ): Flow<Data<Domain>> {
        return flow {
            val networkData = fetchFromNetwork(params)
            saveToMemory(params, networkData)
            saveToStorage(params, networkData)
            emit(Data(content = networkData))
        }.flowOn(ioDispatcher)
            .catch { error ->
                val errorData = Data(content = cachedData, error = error)
                emit(errorData)
            }
    }
```

### Conclusion
This caching mechanism ensures that data retrieval feels like magic, handling all complexity internally while providing a seamless experience to the rest of the app.

