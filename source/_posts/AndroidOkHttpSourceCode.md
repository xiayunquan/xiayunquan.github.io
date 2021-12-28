---
title: OkHttp源码解析
date: 2021-09-24 17:34:27
categories: Android
tags: 源码解析
---

## 一、OkHttp总体架构介绍

#### 简介

OkHttp 是一个处理网络请求的开源项目，是 Android 端最火热的轻量级框架，由Square 公司贡献用于替代 HttpUrlConnection 和 Apache HttpClient。随着 OkHttp 的不断成熟，越来越多的 Android 开发者使用 OkHttp 作为网络框架。OkHttp之所以可以赢得如此多开发者的喜爱，主要得益于如下特点：

- 支持 HTTPS/HTTP2/WebSocket
- 内部维护任务队列线程池，友好支持并发访问 
- 内部维护连接池，支持多路复用，减少连接创建开销
- socket 创建支持最佳路由
- 提供拦截器链（InterceptorChain），实现 request 与 response 的分层处理(如透明GZIP 压缩，logging 等) 

**添加依赖**

```groovy
implementation 'com.squareup.okhttp3:okhttp:3.11.0'
```

> 本篇所有的源码基于OkHttp 3.11.0版本，不同版本源码会有所不同。

#### OkHttp两种调用方式

##### 同步调用

`RealCall.java:68` 

```java
 @Override public Response execute() throws IOException {
    synchronized (this) {
      if (executed) throw new IllegalStateException("Already Executed");
      executed = true;
    }
    captureCallStackTrace();
    eventListener.callStart(this);
    try {
      client.dispatcher().executed(this);
      Response result = getResponseWithInterceptorChain();
      if (result == null) throw new IOException("Canceled");
      return result;
    } catch (IOException e) {
      eventListener.callFailed(this, e);
      throw e;
    } finally {
      client.dispatcher().finished(this);
    }
}
```

首先加锁置标志位，接着使用分配器的 executed 方法将 call 加入到同步队列中，然后调用 getResponseWithInterceptorChain 方法执行 http 请求，最后调用 finishied 方法将 call 从同步队列中删除 。

##### 异步调用

`RealCall.java:93`

```java
@Override public void enqueue(Callback responseCallback) {
    synchronized (this) {
      if (executed) throw new IllegalStateException("Already Executed");
      executed = true;
    }
    captureCallStackTrace();
    eventListener.callStart(this);
    client.dispatcher().enqueue(new AsyncCall(responseCallback));
}
```

同样先加锁置标志位，然后将封装的一个执行体放到异步执行队列中。这里面引入了一个新的类 AsyncCall，这个类继承于 NamedRunnable，实现了 Runnable 接口。NamedRunnable 可以给当前的线程设置名字，并且用模板方法将线程的执行体放到了 execute 方法中。

#### 总体架构

![OkHttp总体架构图](https://img-blog.csdnimg.cn/20210511164224462.png)

OkHttp 的总体架构，大致可以分为以下几层：

- Interface—接口层：接受网络访问请求 
- Protocol—协议层：处理协议逻辑 
- Connection—连接层：管理网络连接，发送新的请求，接收服务器访问
- Cache—缓存层：管理本地缓存 
- I/O—I/O 层：实际数据读写实现 
- Inteceptor—拦截器层：拦截网络访问，插入拦截逻辑

##### Interface——接口层： 

接口层接收用户的网络访问请求（同步请求/异步请求），发起实际的网络访问。OkHttpClient 是 OkHttp 框架的客户端，更确切的说是一个用户面板。用户使用 OkHttp 进行各种设置，发起各种网络请求都是通过 OkHttpClient 完成的。每个 OkHttpClient 内部都维护了属于自己的任务队列，连接池，Cache拦截器等，所以在使用 OkHttp 作为网络框架时应该全局共享一个 OkHttpClient 实例。 

Call 描述一个实际的访问请求，用户的每一个网络请求都是一个 Call 实例。Call 本身只是一个接口，定义了 Call 的接口方法，实际执行过程中，OkHttp 会为每一个请求创建一个 RealCall,每一个 RealCall 内部有一个 AsyncCall: 

```java
final class AsyncCall extends NamedRunnable {
    private final Callback responseCallback;

    AsyncCall(Callback responseCallback) {
        super("OkHttp %s", redactedUrl());
        this.responseCallback = responseCallback;
    }

    String host() {
        return originalRequest.url().host();
    }

    Request request() {
        return originalRequest;
    }

    RealCall get() {
        return RealCall.this;
    }

    @Override
    protected void execute() {...} 
    ...
}
```

AsyncCall 继承的 NamedRunnable 继承自 Runnable 接口：

```java
public abstract class NamedRunnable implements Runnable {
  protected final String name;

  public NamedRunnable(String format, Object... args) {
    this.name = String.format(format, args);
  }

  @Override public final void run() {
    String oldName = Thread.currentThread().getName();
    Thread.currentThread().setName(name);
    try {
      execute();
    } finally {
      Thread.currentThread().setName(oldName);
    }
  }

  protected abstract void execute();
}
```

所以每一个 Call 就是一个线程，而执行 Call 的过程就是执行其 execute 方法的过程。 

Dispatcher 是 OkHttp 的任务队列，其内部维护了一个线程池，当有接收到一个 Call 时，Dispatcher 负责在线程池中找到空闲的线程并执行其 execute 方法。

##### Protocol——协议层：处理协议逻辑 

Protocol 层负责处理协议逻辑，OkHttp 支持 Http1/Http2/WebSocket 协议，并在 3.7 版本中放弃了对 Spdy 协议，鼓励开发者使用 Http2。 

##### Connection——连接层：管理网络连接，发送新的请求，接收服务器访问 

连接层顾名思义就是负责网络连接。在连接层中有一个连接池，统一管理所有的 Socket 连接，当用户新发起一个网络请求时，OkHttp 会首先从连接池中查找是否有符合要求的连接，如果有则直接通过该连接发送网络请求；否则新创建一个网络连接。 

RealConnection 描述一个物理 Socket 连接，连接池中维护多个RealConnection 实例。由于Http2 支持多路复用，一个 RealConnection 可以支持多个网络访问请求，所以 OkHttp 又引入了 StreamAllocation 来描述一个实际的网络请求开销（从逻辑上一个 Stream 对应一个 Call，但在实际网络请求过程中一个 Call 常常涉及到多次请求。如重定向，Authenticate 等场景。所以准确地说，一个 Stream 对应一次请求，而一个 Call 对应一组有逻辑关联的 Stream），一个 RealConnection 对应一个或多个 StreamAllocation，所以 StreamAllocation 可以看做是 RealConenction 的计数器，当 RealConnection 的引用计数变为 0，且长时间没有被其他请求重新占用就将被释放。连接层是 OkHttp 的核心部分，后面再详细介绍。

##### Cache——缓存层：管理本地缓存 

Cache 层负责维护请求缓存，当用户的网络请求在本地已有符合要求的缓存时，OkHttp 会直接从缓存中返回结果，从而节省网络开销。 

##### I/O——I/O 层：实际数据读写实现 

I/O 层负责实际的数据读写。OkHttp 的另一大有点就是其高效的 I/O 操作，这归因于其高效的 I/O 库 okio。 

##### Inteceptor——拦截器层：拦截网络访问，插入拦截逻辑 

拦截器层提供了一个类 AOP 接口，方便用户可以切入到各个层面对网络访问进行拦截并执行相关逻辑。

## 二、OkHttp源码解析

#### 基本使用流程和拦截器

##### 简单示例

```java
OkHttpClient client = new OkHttpClient(); 

Request request = new Request.Builder() .url("https://www.wanandroid.com//hotkey/json") 
.build();

client.newCall(request).enqueue(new Callback() {
    @Override public void onFailure(Call call,IOException e) {
        Log.d("OkHttp","Call Failed:"+e.getMessage());
    }
    @Override public void onResponse(Call call,Response response) throws IOException {
        Log.d("OkHttp","Call succeeded:"+response.message());
    }
});
```

OkHttpClient.newCall 实际是创建一个 RealCall 实例：

```java
@Override public Call newCall(Request request) {
    return new RealCall(this, request);
}
```

RealCall.enqueue 实际就是将一个 RealCall 放入到任务队列中，等待合适的机会执行，最终 RealCall 被转化成一个 AsyncCall 并被放入到任务队列中，AsyncCall 的 execute 方法最终将会被执行：

`RealCall .java:144`

```java
@Override protected void execute() {
      boolean signalledCallback = false;
      try {
        Response response = getResponseWithInterceptorChain(forWebSocket);
        if (canceled) {
          signalledCallback = true;
          responseCallback.onFailure(RealCall.this, new IOException("Canceled"));
        } else {
          signalledCallback = true;
          responseCallback.onResponse(RealCall.this, response);
        }
      } catch (IOException e) {
        if (signalledCallback) {
          // Do not signal the callback twice!
          logger.log(Level.INFO, "Callback failure for " + toLoggableString(), e);
        } else {
          responseCallback.onFailure(RealCall.this, e);
        }
      } finally {
        client.dispatcher().finished(this);
      }
}
```

execute 方法的逻辑并不复杂，简单的说就是： 

- 调用 getResponseWithInterceptorChain 获取服务器返回 
- 通知任务分发器(client.dispatcher)该任务已结束 

getResponseWithInterceptorChain 构建了一个拦截器链，通过依次执行该拦截器链中的每一个拦截器最终得到服务器返回。 

##### 构建拦截器链

首先来看下 getResponseWithInterceptorChain 的实现：

`RealCall.java:158`

```java
Response getResponseWithInterceptorChain() throws IOException {
    // Build a full stack of interceptors.
    List<Interceptor> interceptors = new ArrayList<>();
    interceptors.addAll(client.interceptors());
    interceptors.add(retryAndFollowUpInterceptor);
    interceptors.add(new BridgeInterceptor(client.cookieJar()));
    interceptors.add(new CacheInterceptor(client.internalCache()));
    interceptors.add(new ConnectInterceptor(client));
    if (!forWebSocket) {
      interceptors.addAll(client.networkInterceptors());
    }
    interceptors.add(new CallServerInterceptor(forWebSocket));

    Interceptor.Chain chain = new RealInterceptorChain(interceptors, null, null, null, 0,
        originalRequest, this, eventListener, client.connectTimeoutMillis(),
        client.readTimeoutMillis(), client.writeTimeoutMillis());

    return chain.proceed(originalRequest);
  }
```

其逻辑大致分为两部分： 

- 创建一系列拦截器，并将其放入一个拦截器数组中。这部分拦截器即包括用户自定义的拦截器也包括框架内部拦截器 
- 创建一个拦截器链 RealInterceptorChain，并执行拦截器链的 proceed 方法 

接下来看下 RealInterceptorChain 的实现逻辑： 

```java
public final class RealInterceptorChain implements Interceptor.Chain {
  private final List<Interceptor> interceptors;
  private final StreamAllocation streamAllocation;
  private final HttpCodec httpCodec;
  private final RealConnection connection;
  private final int index;
  private final Request request;
  private final Call call;
  private final EventListener eventListener;
  private final int connectTimeout;
  private final int readTimeout;
  private final int writeTimeout;
  private int calls;

   //省略一部分代码
    ...

  @Override public Response proceed(Request request) throws IOException {
    return proceed(request, streamAllocation, httpCodec, connection);
  }

  public Response proceed(Request request, StreamAllocation streamAllocation, HttpCodec httpCodec,
      RealConnection connection) throws IOException {
    if (index >= interceptors.size()) throw new AssertionError();

    calls++;

    //省略一部分代码
    ...
    
    // Call the next interceptor in the chain.
    RealInterceptorChain next = new RealInterceptorChain(interceptors, streamAllocation, httpCodec,
        connection, index + 1, request, call, eventListener, connectTimeout, readTimeout,
        writeTimeout);
    Interceptor interceptor = interceptors.get(index);
    Response response = interceptor.intercept(next);

    ...
    return response;
  }
}
```

在 proceed 方法中的核心代码可以看到，proceed 实际上也做了两件事： 

- 创建下一个拦截链。传入 index + 1 使得下一个拦截器链只能从下一个拦截器开始访 问


- 执行索引为 index 的 intercept 方法，并将下一个拦截器链传入该方法 

接下来再看下第一个拦截器 RetryAndFollowUpInterceptor 的 intercept 方法： 

`RetryAndFollowUpInterceptor.java:105`

````java
@Override public Response intercept(Chain chain) throws IOException {
    Request request = chain.request();
    RealInterceptorChain realChain = (RealInterceptorChain) chain;
    Call call = realChain.call();
    EventListener eventListener = realChain.eventListener();

    StreamAllocation streamAllocation = new StreamAllocation(client.connectionPool(),
        createAddress(request.url()), call, eventListener, callStackTrace);
    this.streamAllocation = streamAllocation;

    int followUpCount = 0;
    Response priorResponse = null;
    while (true) {
      if (canceled) {
        streamAllocation.release();
        throw new IOException("Canceled");
      }

      Response response;
      boolean releaseConnection = true;
      try {
        response = realChain.proceed(request, streamAllocation, null, null);
        releaseConnection = false;
      } catch (RouteException e) {
        // The attempt to connect via a route failed. The request will not have been sent.
        if (!recover(e.getLastConnectException(), streamAllocation, false, request)) {
          throw e.getFirstConnectException();
        }
        releaseConnection = false;
        continue;
      } catch (IOException e) {
        // An attempt to communicate with a server failed. The request may have been sent.
        boolean requestSendStarted = !(e instanceof ConnectionShutdownException);
        if (!recover(e, streamAllocation, requestSendStarted, request)) throw e;
        releaseConnection = false;
        continue;
      } finally {
        // We're throwing an unchecked exception. Release any resources.
        if (releaseConnection) {
          streamAllocation.streamFailed(null);
          streamAllocation.release();
        }
      }

      // Attach the prior response if it exists. Such responses never have a body.
      if (priorResponse != null) {
        response = response.newBuilder()
            .priorResponse(priorResponse.newBuilder()
                    .body(null)
                    .build())
            .build();
      }

      Request followUp;
      try {
        followUp = followUpRequest(response, streamAllocation.route());
      } catch (IOException e) {
        streamAllocation.release();
        throw e;
      }

      if (followUp == null) {
        if (!forWebSocket) {
          streamAllocation.release();
        }
        return response;
      }

      closeQuietly(response.body());
      ...
    }
}
````

这段代码最关键的代码是: 

```java
response = ((RealInterceptorChain) chain).proceed(request, streamAllocation, null, null); 
```

这行代码就是执行下一个拦截器链的 proceed 方法。而我们知道在下一个拦截器链中又会执行下一个拦截器的 intercept 方法。所以整个执行链就在拦截器与拦截器链中交替执行，最终完成所有拦截器的操作。这也是 OkHttp 拦截器的链式执行逻辑。而一个拦截器的 intercept 方法所执行的逻辑大致分为三部分： 

- 在发起请求前对 request 进行处理 
- 调用下一个拦截器，获取 response 
- 对 response 进行处理，返回给上一个拦截器 

这就是 OkHttp 拦截器机制的核心逻辑。所以一个网络请求实际上就是一个个拦截器执行其intercept 方法的过程。而这其中除了用户自定义的拦截器外还有几个核心拦截器完成了网络访问的核心逻辑，按照先后顺序依次是：

- RetryAndFollowUpInterceptor 


- BridgeInterceptor 
- CacheInterceptor 
- ConnectIntercetot 
- CallServerInterceptor

##### RetryAndFollowUpInterceptor 

如上文代码所示，RetryAndFollowUpInterceptor 负责两部分逻辑： 

- 在网络请求失败后进行重试 
- 当服务器返回当前请求需要进行重定向时直接发起新的请求，并在条件允许情况下复用当前连接

##### BridgeInterceptor 

```java
public final class BridgeInterceptor implements Interceptor {
  private final CookieJar cookieJar;

  public BridgeInterceptor(CookieJar cookieJar) {
    this.cookieJar = cookieJar;
  }

  @Override public Response intercept(Chain chain) throws IOException {
    Request userRequest = chain.request();
    Request.Builder requestBuilder = userRequest.newBuilder();

    RequestBody body = userRequest.body();
    if (body != null) {
      MediaType contentType = body.contentType();
      if (contentType != null) {
        requestBuilder.header("Content-Type", contentType.toString());
      }

      long contentLength = body.contentLength();
      if (contentLength != -1) {
        requestBuilder.header("Content-Length", Long.toString(contentLength));
        requestBuilder.removeHeader("Transfer-Encoding");
      } else {
        requestBuilder.header("Transfer-Encoding", "chunked");
        requestBuilder.removeHeader("Content-Length");
      }
    }

    if (userRequest.header("Host") == null) {
      requestBuilder.header("Host", hostHeader(userRequest.url(), false));
    }

    if (userRequest.header("Connection") == null) {
      requestBuilder.header("Connection", "Keep-Alive");
    }

    // If we add an "Accept-Encoding: gzip" header field we're responsible for also decompressing
    // the transfer stream.
    boolean transparentGzip = false;
    if (userRequest.header("Accept-Encoding") == null && userRequest.header("Range") == null) {
      transparentGzip = true;
      requestBuilder.header("Accept-Encoding", "gzip");
    }

    List<Cookie> cookies = cookieJar.loadForRequest(userRequest.url());
    if (!cookies.isEmpty()) {
      requestBuilder.header("Cookie", cookieHeader(cookies));
    }

    if (userRequest.header("User-Agent") == null) {
      requestBuilder.header("User-Agent", Version.userAgent());
    }

    Response networkResponse = chain.proceed(requestBuilder.build());

    HttpHeaders.receiveHeaders(cookieJar, userRequest.url(), networkResponse.headers());

    Response.Builder responseBuilder = networkResponse.newBuilder()
        .request(userRequest);

    if (transparentGzip
        && "gzip".equalsIgnoreCase(networkResponse.header("Content-Encoding"))
        && HttpHeaders.hasBody(networkResponse)) {
      GzipSource responseBody = new GzipSource(networkResponse.body().source());
      Headers strippedHeaders = networkResponse.headers().newBuilder()
          .removeAll("Content-Encoding")
          .removeAll("Content-Length")
          .build();
      responseBuilder.headers(strippedHeaders);
      String contentType = networkResponse.header("Content-Type");
      responseBuilder.body(new RealResponseBody(contentType, -1L, Okio.buffer(responseBody)));
    }

    return responseBuilder.build();
  }

  /** Returns a 'Cookie' HTTP request header with all cookies, like {@code a=b; c=d}. */
  private String cookieHeader(List<Cookie> cookies) {
    StringBuilder cookieHeader = new StringBuilder();
    for (int i = 0, size = cookies.size(); i < size; i++) {
      if (i > 0) {
        cookieHeader.append("; ");
      }
      Cookie cookie = cookies.get(i);
      cookieHeader.append(cookie.name()).append('=').append(cookie.value());
    }
    return cookieHeader.toString();
  }
}
```

BridgeInterceptor 主要负责以下几部分内容： 

- 设置内容长度，内容编码 
- 设置 gzip 压缩，并在接收到内容后进行解压。省去了应用层处理数据解压的麻烦 
- 添加 cookie 
- 设置其他报头，如 User-Agent,Host,Keep-alive 等。其中 Keep-Alive 是实现多路复用的必要步骤

##### CacheInterceptor

```java
public final class CacheInterceptor implements Interceptor {
  final InternalCache cache;

  public CacheInterceptor(InternalCache cache) {
    this.cache = cache;
  }

  @Override public Response intercept(Chain chain) throws IOException {
    //尝试获取缓存
    Response cacheCandidate = cache != null
        ? cache.get(chain.request())
        : null;

    long now = System.currentTimeMillis();

    //获取缓存策略
    CacheStrategy strategy = new CacheStrategy.Factory(now, chain.request(), cacheCandidate).get();
    Request networkRequest = strategy.networkRequest;
    Response cacheResponse = strategy.cacheResponse;
    //如果有缓存，更新下相关统计指标：命中率
    if (cache != null) {
      cache.trackResponse(strategy);
    }

    //如果当前缓存不符合要求，将其 close
    if (cacheCandidate != null && cacheResponse == null) {
      closeQuietly(cacheCandidate.body()); // The cache candidate wasn't applicable. Close it.
    }

    // If we're forbidden from using the network and the cache is insufficient, fail.(如果不能使用网络，同时又没有符合条件的缓存，直接抛 504 错误)
    if (networkRequest == null && cacheResponse == null) {
      return new Response.Builder()
          .request(chain.request())
          .protocol(Protocol.HTTP_1_1)
          .code(504)
          .message("Unsatisfiable Request (only-if-cached)")
          .body(Util.EMPTY_RESPONSE)
          .sentRequestAtMillis(-1L)
          .receivedResponseAtMillis(System.currentTimeMillis())
          .build();
    }

    // If we don't need the network, we're done.(如果有缓存同时又不使用网络，则直接返回缓存结果)
    if (networkRequest == null) {
      return cacheResponse.newBuilder()
          .cacheResponse(stripBody(cacheResponse))
          .build();
    }

    Response networkResponse = null;
    try {
      networkResponse = chain.proceed(networkRequest);
    } finally {
      // If we're crashing on I/O or otherwise, don't leak the cache body.
      if (networkResponse == null && cacheCandidate != null) {
        closeQuietly(cacheCandidate.body());
      }
    }

    // If we have a cache response too, then we're doing a conditional get.(如果既有缓存，同时又发起了请求，说明此时是一个 Conditional Get 请求)
    if (cacheResponse != null) {
      if (networkResponse.code() == HTTP_NOT_MODIFIED) {
        Response response = cacheResponse.newBuilder()
            .headers(combine(cacheResponse.headers(), networkResponse.headers()))
            .sentRequestAtMillis(networkResponse.sentRequestAtMillis())
            .receivedResponseAtMillis(networkResponse.receivedResponseAtMillis())
            .cacheResponse(stripBody(cacheResponse))
            .networkResponse(stripBody(networkResponse))
            .build();
        networkResponse.body().close();

        // Update the cache after combining headers but before stripping the
        // Content-Encoding header (as performed by initContentStream()).
        cache.trackConditionalCacheHit();
        cache.update(cacheResponse, response);
        return response;
      } else {
        // 如果响应资源有更新，关掉原有缓存
        closeQuietly(cacheResponse.body());
      }
    }

    Response response = networkResponse.newBuilder()
        .cacheResponse(stripBody(cacheResponse))
        .networkResponse(stripBody(networkResponse))
        .build();

    if (cache != null) {
      if (HttpHeaders.hasBody(response) && CacheStrategy.isCacheable(response, networkRequest)) {
        // Offer this request to the cache.(将网络响应写入 cache 中)
        CacheRequest cacheRequest = cache.put(response);
        return cacheWritingResponse(cacheRequest, response);
      }

      if (HttpMethod.invalidatesCache(networkRequest.method())) {
        try {
          cache.remove(networkRequest);
        } catch (IOException ignored) {
          // The cache cannot be written.
        }
      }
    }

    return response;
  }
  ...
}
```

CacheInterceptor 的职责很明确，就是负责 Cache 的管理 

- 当网络请求有符合要求的 Cache 时直接返回 Cache 
- 当服务器返回内容有改变时更新当前 cache
- 如果当前 cache 失效，则删除改缓存

##### ConnectInterceptor

```java
public final class ConnectInterceptor implements Interceptor {
  public final OkHttpClient client;

  public ConnectInterceptor(OkHttpClient client) {
    this.client = client;
  }

  @Override public Response intercept(Chain chain) throws IOException {
    RealInterceptorChain realChain = (RealInterceptorChain) chain;
    Request request = realChain.request();
    StreamAllocation streamAllocation = realChain.streamAllocation();

    // We need the network to satisfy this request. Possibly for validating a conditional GET.
    boolean doExtensiveHealthChecks = !request.method().equals("GET");
    HttpCodec httpCodec = streamAllocation.newStream(client, chain, doExtensiveHealthChecks);
    RealConnection connection = streamAllocation.connection();

    return realChain.proceed(request, streamAllocation, httpCodec, connection);
  }
}
```

ConnectInterceptor 的 intercept 方法只有一行关键代码: 

```
RealConnection connection = streamAllocation.connection(); 
```

即为当前请求找到合适的连接，可能复用已有连接也可能是重新创建的连接，返回的连接由连接池负责决定。 

##### CallServerInterceptor

```java
public final class CallServerInterceptor implements Interceptor {
  private final boolean forWebSocket;

  public CallServerInterceptor(boolean forWebSocket) {
    this.forWebSocket = forWebSocket;
  }

  @Override public Response intercept(Chain chain) throws IOException {
    RealInterceptorChain realChain = (RealInterceptorChain) chain;
    HttpCodec httpCodec = realChain.httpStream();
    StreamAllocation streamAllocation = realChain.streamAllocation();
    RealConnection connection = (RealConnection) realChain.connection();
    Request request = realChain.request();

    long sentRequestMillis = System.currentTimeMillis();

    realChain.eventListener().requestHeadersStart(realChain.call());
    httpCodec.writeRequestHeaders(request);
    realChain.eventListener().requestHeadersEnd(realChain.call(), request);

    Response.Builder responseBuilder = null;
    ...

    httpCodec.finishRequest();

    if (responseBuilder == null) {
      realChain.eventListener().responseHeadersStart(realChain.call());
      responseBuilder = httpCodec.readResponseHeaders(false);
    }

    Response response = responseBuilder
        .request(request)
        .handshake(streamAllocation.connection().handshake())
        .sentRequestAtMillis(sentRequestMillis)
        .receivedResponseAtMillis(System.currentTimeMillis())
        .build();

    int code = response.code();
    if (code == 100) {
      // server sent a 100-continue even though we did not request one.
      // try again to read the actual response
      responseBuilder = httpCodec.readResponseHeaders(false);

      response = responseBuilder
              .request(request)
              .handshake(streamAllocation.connection().handshake())
              .sentRequestAtMillis(sentRequestMillis)
              .receivedResponseAtMillis(System.currentTimeMillis())
              .build();

      code = response.code();
    }

    realChain.eventListener()
            .responseHeadersEnd(realChain.call(), response);

    if (forWebSocket && code == 101) {
      // Connection is upgrading, but we need to ensure interceptors see a non-null response body.
      response = response.newBuilder()
          .body(Util.EMPTY_RESPONSE)
          .build();
    } else {
      response = response.newBuilder()
          .body(httpCodec.openResponseBody(response))
          .build();
    }

    if ("close".equalsIgnoreCase(response.request().header("Connection"))
        || "close".equalsIgnoreCase(response.header("Connection"))) {
      streamAllocation.noNewStreams();
    }

    if ((code == 204 || code == 205) && response.body().contentLength() > 0) {
      throw new ProtocolException(
          "HTTP " + code + " had non-zero Content-Length: " + response.body().contentLength());
    }

    return response;
  }
  ...
}
```

CallServerInterceptor 负责向服务器发起真正的访问请求，并在接收到服务器返回后读取响应返回。

##### 整体流程

整个网络访问的核心步骤，总结起来如下图所示： 

![OkHttp interceptor](https://img-blog.csdnimg.cn/20210512095342458.png)



#### 任务队列

OkHttp 的任务队列在内部维护了一个线程池用于执行具体的网络请求。而线程池最大的好处在于通过线程复用减少非核心任务的损耗。

##### 线程池的优点

多线程技术主要解决处理器单元内多个线程执行的问题，它可以显著减少处 

理器单元的闲置时间，增加处理器单元的吞吐能力。但如果对多线程应用不 

当，会增加对单个任务的处理时间。可以举一个简单的例子： 

假设在一台服务器完成一项任务的时间为 T ，

T1 创建线程的时间 ；

T2 在线程中执行任务的时间，包括线程间同步所需时间 ；

T3 线程销毁的时间 ；

显然 T ＝ T1＋T2＋T3。注意这是一个极度简化的假设。 

可以看出 T1、T3 是多线程本身带来的开销（在 Java 中，通过映射 pThead， 并进一步通过SystemCall 实现 native 线程），我们渴望减少 T1、T3 所用的时间，从而减少 T 的时间。但一些线程的使用者并没有注意到这一点，所以在程序中频繁的创建或销毁线程，这导致 T1 和 T3 在 T 中占有相当比例。

线程池技术正是关注如何缩短或调整 T1，T3 时间的技术，从而提高服务器程序性能的。

- 通过对线程进行缓存，减少了创建销毁的时间损失
- 通过控制线程数量阀值，减少了当线程过少时带来的 CPU 闲置（比如说长时间卡在I/O 上了）与线程过多时对 JVM 的内存与线程切换时系统调用的压力。

类似的还有 Socket 连接池、DB 连接池、CommonPool(比如 Jedis)等技术。

OkHttp 的任务队列主要由两部分组成： 

- 任务分发器 dispatcher：负责为任务找到合适的执行线程 
- 网络请求任务线程池 

```java
public final class Dispatcher {
  private int maxRequests = 64;
  private int maxRequestsPerHost = 5;
  private @Nullable Runnable idleCallback;

  /** Executes calls. Created lazily. */
  private @Nullable ExecutorService executorService;

  /** Ready async calls in the order they'll be run. */
  private final Deque<AsyncCall> readyAsyncCalls = new ArrayDeque<>();

  /** Running asynchronous calls. Includes canceled calls that haven't finished yet. */
  private final Deque<AsyncCall> runningAsyncCalls = new ArrayDeque<>();

  /** Running synchronous calls. Includes canceled calls that haven't finished yet. */
  private final Deque<RealCall> runningSyncCalls = new ArrayDeque<>();

  public Dispatcher(ExecutorService executorService) {
    this.executorService = executorService;
  }

  public Dispatcher() {
  }

  public synchronized ExecutorService executorService() {
    if (executorService == null) {
      executorService = new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60, TimeUnit.SECONDS,
          new SynchronousQueue<Runnable>(), Util.threadFactory("OkHttp Dispatcher", false));
    }
    return executorService;
  }
  ...
}
```

参数说明如下： 

- readyAsyncCalls：待执行异步任务队列 
- runningAsyncCalls：运行中异步任务队列 
- runningSyncCalls：运行中同步任务队列 
- executorService：任务队列线程池：

```java
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             threadFactory, defaultHandler);
    }
}
```

- int corePoolSize: 最小并发线程数，这里并发同时包括空闲与活动的线 

程，如果是 0 的话，空闲一段时间后所有线程将全部被销毁 

- int maximumPoolSize: 最大线程数，当任务进来时可以扩充的线程最大值，当大于了这个值就会根据丢弃处理机制来处理 
- long keepAliveTime: 当线程数大于 corePoolSize 时，多余的空闲线程的最大存活时间，类似于 HTTP 中的 Keep-alive 
- TimeUnit unit: 时间单位，一般用秒 
- BlockingQueue workQueue: 工作队列，先进先出
- ThreadFactory threadFactory: 单个线程的工厂，可以打 Log，设置 Daemon(即当 JVM 退出时，线程自动结束)等 

在 OkHttp 中，构建了一个阀值为[0, Integer.MAX_VALUE]的线程池，它不保留任何最小线程数，随时创建更多的线程数，当线程空闲时只能活 60 秒，它使用了一个不存储元素的阻塞工作队列，一个叫做"OkHttp Dispatcher"的线程工厂。 

也就是说，在实际运行中，当收到 10 个并发请求时，线程池会创建十个线 程，当工作完成后，线程池会在 60s 后相继关闭所有线程。

##### Dispatcher 分发器

dispatcher 分发器类似于 Ngnix 中的反向代理，通过 Dispatcher 将任务分发到合适的空闲线程，实现非阻塞，高可用，高并发连接。

![dispatcher](https://img-blog.csdnimg.cn/20210512104614361.png)

##### 同步请求

当我们使用 OkHttp 进行同步请求时，一般构造如下：

```java
OkHttpClient client = new OkHttpClient(); 

Request request = new Request.Builder() .url("https://www.wanandroid.com//hotkey/json") 
.build();

Response response = client.newCall(request).execute();
```

```java
@Override public Response execute() throws IOException {
    ...
    try {
      client.dispatcher().executed(this);
      Response result = getResponseWithInterceptorChain();
      if (result == null) throw new IOException("Canceled");
      return result;
    } catch (IOException e) {
      eventListener.callFailed(this, e);
      throw e;
    } finally {
      client.dispatcher().finished(this);
    }
}
```

同步请求的执行逻辑是： 

- 将对应任务加入分发器 
- 执行任务 
- 执行完成后通知 dispatcher 对应任务已完成，对应任务出队 



##### 异步请求

```java
OkHttpClient client = new OkHttpClient(); 

Request request = new Request.Builder() .url("https://www.wanandroid.com//hotkey/json") 
.build();

client.newCall(request).enqueue(new Callback() {
    @Override public void onFailure(Call call,IOException e) {
        Log.d("OkHttp","Call Failed:"+e.getMessage());
    }
    @Override public void onResponse(Call call,Response response) throws IOException {
        Log.d("OkHttp","Call succeeded:"+response.message());
    }
});
```

当 OkHttpClient 的请求入队时，根据代码，我们可以发现实际上是Dispatcher 进行了入队操作。

`Dispatcher.java:129`

```java
synchronized void enqueue(AsyncCall call) {
    if (runningAsyncCalls.size() < maxRequests && runningCallsForHost(call) < maxRequestsPerHost) {
      //添加正在运行的请求
      runningAsyncCalls.add(call);
      
      //线程池执行请求
      executorService().execute(call);
    } else {
      //添加到缓存队列排队等待
      readyAsyncCalls.add(call);
    }
}
```

如果满足条件：  

- 当前请求数小于最大请求数（64）  
- 对单一 host 的请求小于阈值（5）

将该任务插入正在执行任务队列，并执行对应任务。如果不满足则将其放入待执行队列。 

接下来看看 AsyncCall的execute方法

```java
@Override protected void execute() {
      boolean signalledCallback = false;
      try {
        //执行I/O耗时任务
        Response response = getResponseWithInterceptorChain();
        if (retryAndFollowUpInterceptor.isCanceled()) {
          signalledCallback = true;
          //回调,这里回调是在线程池中
          responseCallback.onFailure(RealCall.this, new IOException("Canceled"));
        } else {
          signalledCallback = true;
          //回调,这里回调是在线程池中
          responseCallback.onResponse(RealCall.this, response);
        }
      } catch (IOException e) {
        if (signalledCallback) {
          // Do not signal the callback twice!
          Platform.get().log(INFO, "Callback failure for " + toLoggableString(), e);
        } else {
          eventListener.callFailed(RealCall.this, e);
          responseCallback.onFailure(RealCall.this, e);
        }
      } finally {
        //通知分发器相关任务 已结束
        client.dispatcher().finished(this);
      }
}
```

`Dispatcher.java:198`

```java
private <T> void finished(Deque<T> calls, T call, boolean promoteCalls) {
    int runningCallsCount;
    Runnable idleCallback;
    synchronized (this) {
      if (!calls.remove(call)) throw new AssertionError("Call wasn't in-flight!");
      if (promoteCalls) promoteCalls();
      runningCallsCount = runningCallsCount();
      idleCallback = this.idleCallback;
    }

    if (runningCallsCount == 0 && idleCallback != null) {
      idleCallback.run();
    }
}
```

- 空闲出多余线程，调用 promoteCalls 调用待执行的任务
- 如果当前整个线程池都空闲下来，执行空闲通知回调线程(idleCallback) 

```java
private void promoteCalls() {
    if (runningAsyncCalls.size() >= maxRequests) return; // Already running max capacity.
    if (readyAsyncCalls.isEmpty()) return; // No ready calls to promote.

    for (Iterator<AsyncCall> i = readyAsyncCalls.iterator(); i.hasNext(); ) {
      AsyncCall call = i.next();

      if (runningCallsForHost(call) < maxRequestsPerHost) {
        i.remove();
        runningAsyncCalls.add(call);
        executorService().execute(call);
      }

      if (runningAsyncCalls.size() >= maxRequests) return; // Reached max capacity.
    }
}
```

promoteCalls 的逻辑也很简单：扫描待执行任务队列，将任务放入正在执行任务队列，并执行该任务

##### 总结

以上就是整个任务队列的实现细节，总结起来有以下几个特点： 

- OkHttp 采用 Dispatcher 技术，类似于 Nginx，与线程池配合实现了高并发，低阻塞的运行 
- Okhttp 采用 Deque 作为缓存，按照入队的顺序先进先出 
- OkHttp 最出彩的地方就是在 try/finally 中调用了 finished 函数，可以主动控制等待队列的移动，而不是采用锁或者 wait/notify，极大减少了编码复杂性

#### 缓存策略

##### HTTP 协议中缓存部分的相关域

合理地利用本地缓存可以有效地减少网络开销，减少响应延迟。HTTP 报头也定义了很多与缓存有关的域来控制缓存。

首先来了解下 HTTP 协议中缓存部分的相关域。

**Expires** 

超时时间，一般用在服务器的 response 报头中用于告知客户端对应资源的过期时间。当客户端需要再次请求相同资源时先比较其过期时间，如果尚未超过过期时间则直接返回缓存结果，如果已经超过则重新请求。

**Cache-Control**

相对值，单位时秒，表示当前资源的有效期。Cache-Control 比 Expires 优先级更高：

**Last-Modified-Date**

客户端第一次请求时，服务器返回： 

> Last-Modified: Tue, 12 Jan 2021 09:31:27 GMT 

当客户端二次请求时，可以头部加上如下 header: 

> If-Modified-Since: Tue, 12 Jan 2021 09:31:27 GMT 

如果当前资源没有被二次修改，服务器返回 304 告知客户端直接复用本地缓存。 

**ETag** 

ETag 是对资源文件的一种摘要，可以通过 ETag 值来判断文件是否有修改。当客户端第一次 

请求某资源时，服务器返回： 

> ETag: "5694c7ef-24dc" 

客户端再次请求时，可在头部加上如下域： 

> If-None-Match: "5694c7ef-24dc" 

如果文件并未改变，则服务器返回 304 告知客户端可以复用本地缓存。 

**no-cache/no-store** 

不使用缓存 

**only-if-cached** 

只使用缓存

##### Cache 源码分析

OkHttp 的缓存工作都是在 CacheInterceptor 中完成的，Cache 部分有如下几个关键类：

- Cache：Cache 管理器，其内部包含一个 DiskLruCache 将 cache 写入文件系统，Cache 内部通过 requestCount、networkCount、hitCount 三个统计指标来优化缓存效率 
- CacheStrategy：缓存策略，其内部维护一个 request 和 response，通过指定 request 和 response 来描述是通过网络还是缓存获取response，或二者同时使用
- CacheStrategy$Factory：缓存策略工厂类根据实际请求返回对应的缓存策略既然实际的缓存工作都是在 CacheInterceptor 中完成的。

通过上面拦截器的分析CacheInterceptor类的代码可以看出，所有的动作都是以 CacheStrategy 缓存策略为依据做出的，那么来看下缓存策略是如何生成的，相关代码实现在 CacheStrategy$Factory.get()方法中：

```java
public CacheStrategy get() {
      CacheStrategy candidate = getCandidate();

      if (candidate.networkRequest != null && request.cacheControl().onlyIfCached()) {
        // We're forbidden from using the network and the cache is insufficient.
        return new CacheStrategy(null, null);
      }

      return candidate;
}

    /** Returns a strategy to use assuming the request can use the network. */
private CacheStrategy getCandidate() {
      // No cached response.(若本地没有缓存，发起网络请求)
      if (cacheResponse == null) {
        return new CacheStrategy(request, null);
      }

      // Drop the cached response if it's missing a required handshake.(如果当前请求是 HTTPS，而缓存没有 TLS 握手，重新发起网络请求)
      if (request.isHttps() && cacheResponse.handshake() == null) {
        return new CacheStrategy(request, null);
      }

      // If this response shouldn't have been stored, it should never be used
      // as a response source. This check should be redundant as long as the
      // persistence store is well-behaved and the rules are constant.
      if (!isCacheable(cacheResponse, request)) {
        return new CacheStrategy(request, null);
      }
  	  //如果当前的缓存策略是不缓存或者是 conditional get，发起网络请求
      CacheControl requestCaching = request.cacheControl();
      if (requestCaching.noCache() || hasConditions(request)) {
        return new CacheStrategy(request, null);
      }

      CacheControl responseCaching = cacheResponse.cacheControl();
      if (responseCaching.immutable()) {
        return new CacheStrategy(null, cacheResponse);
      }
	  //缓存 age
      long ageMillis = cacheResponseAge();
      long freshMillis = computeFreshnessLifetime();

      if (requestCaching.maxAgeSeconds() != -1) {
        freshMillis = Math.min(freshMillis, SECONDS.toMillis(requestCaching.maxAgeSeconds()));
      }

      long minFreshMillis = 0;
      if (requestCaching.minFreshSeconds() != -1) {
        minFreshMillis = SECONDS.toMillis(requestCaching.minFreshSeconds());
      }

      long maxStaleMillis = 0;
      if (!responseCaching.mustRevalidate() && requestCaching.maxStaleSeconds() != -1) {
        maxStaleMillis = SECONDS.toMillis(requestCaching.maxStaleSeconds());
      }

      if (!responseCaching.noCache() && ageMillis + minFreshMillis < freshMillis + maxStaleMillis) {
        Response.Builder builder = cacheResponse.newBuilder();
        //虽然缓存过期了，但是缓存还可以使用，则在头部添加 110 警告码
        if (ageMillis + minFreshMillis >= freshMillis) {
          builder.addHeader("Warning", "110 HttpURLConnection \"Response is stale\"");
        }
        long oneDayMillis = 24 * 60 * 60 * 1000L;
        if (ageMillis > oneDayMillis && isFreshnessLifetimeHeuristic()) {
          builder.addHeader("Warning", "113 HttpURLConnection \"Heuristic expiration\"");
        }
        return new CacheStrategy(null, builder.build());
      }

      // Find a condition to add to the request. If the condition is satisfied, the response body
      // will not be transmitted.
      String conditionName;
      String conditionValue;
      if (etag != null) {
        conditionName = "If-None-Match";
        conditionValue = etag;
      } else if (lastModified != null) {
        conditionName = "If-Modified-Since";
        conditionValue = lastModifiedString;
      } else if (servedDate != null) {
        conditionName = "If-Modified-Since";
        conditionValue = servedDateString;
      } else {
        return new CacheStrategy(request, null); // No condition! Make a regular request.
      }

      Headers.Builder conditionalRequestHeaders = request.headers().newBuilder();
      Internal.instance.addLenient(conditionalRequestHeaders, conditionName, conditionValue);

      Request conditionalRequest = request.newBuilder()
          .headers(conditionalRequestHeaders.build())
          .build();
      return new CacheStrategy(conditionalRequest, cacheResponse);
}
```

可以看到其核心逻辑在 getCandidate 函数中。基本就是 HTTP 缓存协议的实现。

##### DiskLruCache

Cache 内部通过 DiskLruCache 管理 cache 在文件系统层面的创建，读取，清理等等工作， 

接下来看下 DiskLruCache 的主要逻辑：

**journalFile**

DiskLruCache 内部日志文件，对 cache 的每一次读写都对应一条日志记录，DiskLruCache 

通过分析日志分析和创建 cache。日志文件格式如下：

```
libcore.io.DiskLruCache 
1
100 
2

CLEAN 3400330d1dfc7f3f7f4b8d4d803dfcf6 832 21054
DIRTY 335c4c6028171cfddfbaae1a9c313c52 
CLEAN 335c4c6028171cfddfbaae1a9c313c52 3934 2342 
REMOVE 335c4c6028171cfddfbaae1a9c313c52 
DIRTY 1ab96a171faeeee38496d8b330771a7a 
CLEAN 1ab96a171faeeee38496d8b330771a7a 1600 234 
READ 335c4c6028171cfddfbaae1a9c313c52 
READ 3400330d1dfc7f3f7f4b8d4d803dfcf6
```

前 5 行固定不变，分别为：常量libcore.io.DiskLruCache、diskCache 版本、应用程序版本、valueCount(后文介绍)、空行 

接下来每一行对应一个 cache entry 的一次状态记录，其格式为：[状态（DIRTY,CLEAN,READ,REMOVE），key，状态相关 value(可选)]: 

\- DIRTY:表明一个 cache entry 正在被创建或更新，每一个成功的 DIRTY 记录都应该对应一个 CLEAN 或 REMOVE 操作。如果一个 DIRTY 缺少预期匹配的 CLEAN/REMOVE，则对应 entry 操作失败，需要将其从 lruEntries 中删除 

\- CLEAN:说明 cache 已经被成功操作，当前可以被正常读取。每一个 CLEAN 行还需要记 录其每一个 value 的长度 

\- READ: 记录一次 cache 读取操作 

\- REMOVE:记录一次 cache 清除

日志文件的应用场景主要有四个： 

- DiskCacheLru 初始化时通过读取日志文件创建 cache 容器lruEntries。同时通过日志过滤操作不成功的 cache 项。相关逻辑在 DiskLruCache.readJournalLine, DiskLruCache.processJournal 
- 初始化完成后，为避免日志文件不断膨胀，对日志进行重建精简，具体逻辑在 DiskLruCache.rebuildJournal 
- 每当有 cache 操作时将其记录入日志文件中以备下次初始化时使用 
- 当冗余日志过多时，通过调用 cleanUpRunnable 线程重建日志

**DiskLruCache.Entry** 

每一个 DiskLruCache.Entry 对应一个 cache 记录：

```java
private final class Entry {
    final String key;

    /** Lengths of this entry's files. */
    final long[] lengths;
    final File[] cleanFiles;
    final File[] dirtyFiles;

    /** True if this entry has ever been published. */
    boolean readable;

    /** The ongoing edit or null if this entry is not being edited. */
    Editor currentEditor;

    /** The sequence number of the most recently committed edit to this entry. */
    long sequenceNumber;

    Entry(String key) {
      this.key = key;

      lengths = new long[valueCount];
      cleanFiles = new File[valueCount];
      dirtyFiles = new File[valueCount];

      // The names are repetitive so re-use the same builder to avoid allocations.
      StringBuilder fileBuilder = new StringBuilder(key).append('.');
      int truncateTo = fileBuilder.length();
      for (int i = 0; i < valueCount; i++) {
        fileBuilder.append(i);
        cleanFiles[i] = new File(directory, fileBuilder.toString());
        fileBuilder.append(".tmp");
        dirtyFiles[i] = new File(directory, fileBuilder.toString());
        fileBuilder.setLength(truncateTo);
      }
    }

    /** Set lengths using decimal numbers like "10123". */
    void setLengths(String[] strings) throws IOException {
      if (strings.length != valueCount) {
        throw invalidLengths(strings);
      }

      try {
        for (int i = 0; i < strings.length; i++) {
          lengths[i] = Long.parseLong(strings[i]);
        }
      } catch (NumberFormatException e) {
        throw invalidLengths(strings);
      }
    }

    /** Append space-prefixed lengths to {@code writer}. */
    void writeLengths(BufferedSink writer) throws IOException {
      for (long length : lengths) {
        writer.writeByte(' ').writeDecimalLong(length);
      }
    }

    private IOException invalidLengths(String[] strings) throws IOException {
      throw new IOException("unexpected journal line: " + Arrays.toString(strings));
    }

    /**
     * Returns a snapshot of this entry. This opens all streams eagerly to guarantee that we see a
     * single published snapshot. If we opened streams lazily then the streams could come from
     * different edits.
     */
    Snapshot snapshot() {
      if (!Thread.holdsLock(DiskLruCache.this)) throw new AssertionError();

      Source[] sources = new Source[valueCount];
      long[] lengths = this.lengths.clone(); // Defensive copy since these can be zeroed out.
      try {
        for (int i = 0; i < valueCount; i++) {
          sources[i] = fileSystem.source(cleanFiles[i]);
        }
        return new Snapshot(key, sequenceNumber, sources, lengths);
      } catch (FileNotFoundException e) {
        // A file must have been deleted manually!
        for (int i = 0; i < valueCount; i++) {
          if (sources[i] != null) {
            Util.closeQuietly(sources[i]);
          } else {
            break;
          }
        }
        // Since the entry is no longer valid, remove it so the metadata is accurate (i.e. the cache
        // size.)
        try {
          removeEntry(this);
        } catch (IOException ignored) {
        }
        return null;
      }
    }
}
```

一个 Entry 主要由以下几部分构成： 

- key：每个 cache 都有一个 key 作为其标识符。当前 cache 的 key 为其对应 URL 的MD5 字符串 
- cleanFiles/dirtyFiles：每一个 Entry 对应多个文件，其对应的文件数由 DiskLruCache.valueCount 指定。当前在 OkHttp 中 valueCount 为 2。即每个 cache 对应 2 个 cleanFiles，2 个 dirtyFiles。其中第一个 cleanFiles/dirtyFiles 记录 cache 的 meta 数据（如 URL,创建时间，SSL 握手记录等等），第二个文件记录 cache 的真正内容。cleanFiles 记录处于稳定状态的 cache 结果，dirtyFiles 记录处于创建或更新状态的 cache 
- currentEditor：entry 编辑器，对 entry 的所有操作都是通过其编辑器完成。编辑器内部添加了同步锁

**cleanupRunnable**

清理线程，用于重建精简日志：

```java
private final Runnable cleanupRunnable = new Runnable() {
    public void run() {
      synchronized (DiskLruCache.this) {
        if (!initialized | closed) {
          return; // Nothing to do
        }

        try {
          trimToSize();
        } catch (IOException ignored) {
          mostRecentTrimFailed = true;
        }

        try {
          if (journalRebuildRequired()) {
            rebuildJournal();
            redundantOpCount = 0;
          }
        } catch (IOException e) {
          mostRecentRebuildFailed = true;
          journalWriter = Okio.buffer(Okio.blackhole());
        }
      }
    }
};
```

其触发条件在 journalRebuildRequired()方法中：当冗余日志超过日志文件本身的一般且总条数超过 2000 时执行 

```java
boolean journalRebuildRequired() {
    final int redundantOpCompactThreshold = 2000;
    return redundantOpCount >= redundantOpCompactThreshold
        && redundantOpCount >= lruEntries.size();
}
```

**SnapShot**

cache 快照，记录了特定 cache 在某一个特定时刻的内容。每次向 DiskLruCache 请求时返回的都是目标 cache 的一个快照,相关逻辑在 DiskLruCache.get 中：

```java
public synchronized Snapshot get(String key) throws IOException {
    initialize();

    checkNotClosed();
    validateKey(key);
    Entry entry = lruEntries.get(key);
    if (entry == null || !entry.readable) return null;

    Snapshot snapshot = entry.snapshot();
    if (snapshot == null) return null;

    redundantOpCount++;
    journalWriter.writeUtf8(READ).writeByte(' ').writeUtf8(key).writeByte('\n');
    if (journalRebuildRequired()) {
      executor.execute(cleanupRunnable);
    }

    return snapshot;
}
```

**lruEntries** 

管理 cache entry 的容器，其数据结构是 LinkedHashMap。通过 LinkedHashMap 本身的实现逻辑达到 cache 的 LRU 替换 

**FileSystem** 

使用 Okio 对 File 的封装，简化了 I/O 操作。 

**DiskLruCache.edit** 

DiskLruCache 可以看成是 Cache 在文件系统层的具体实现，所以其基本操作接口存在一一 对应的关系： 

- Cache.get() —>DiskLruCache.get() 
- Cache.put()—>DiskLruCache.edit() //cache 插入 
- Cache.remove()—>DiskLruCache.remove() 
- Cache.update()—>DiskLruCache.edit()//cache 更新 

Cache.put方法:

```java
@Nullable CacheRequest put(Response response) {
    String requestMethod = response.request().method();

    if (HttpMethod.invalidatesCache(response.request().method())) {
      try {
        remove(response.request());
      } catch (IOException ignored) {
        // The cache cannot be written.
      }
      return null;
    }
    if (!requestMethod.equals("GET")) {
      // Don't cache non-GET responses. We're technically allowed to cache
      // HEAD requests and some POST requests, but the complexity of doing
      // so is high and the benefit is low.
      return null;
    }

    if (HttpHeaders.hasVaryAll(response)) {
      return null;
    }

    Entry entry = new Entry(response);
    DiskLruCache.Editor editor = null;
    try {
      editor = cache.edit(key(response.request().url()));
      if (editor == null) {
        return null;
      }
      entry.writeTo(editor);
      return new CacheRequestImpl(editor);
    } catch (IOException e) {
      abortQuietly(editor);
      return null;
    }
}
```

可以看到核心逻辑在 editor = cache.edit(key(response.request().url()));,相关代码在 DiskLruCache.edit:

```java
synchronized Editor edit(String key, long expectedSequenceNumber) throws IOException {
    initialize();

    checkNotClosed();
    validateKey(key);
    Entry entry = lruEntries.get(key);
    if (expectedSequenceNumber != ANY_SEQUENCE_NUMBER && (entry == null
        || entry.sequenceNumber != expectedSequenceNumber)) {
      return null; // Snapshot is stale.
    }
    if (entry != null && entry.currentEditor != null) {
      return null; // Another edit is in progress.
    }
    if (mostRecentTrimFailed || mostRecentRebuildFailed) {
      // The OS has become our enemy! If the trim job failed, it means we are storing more data than
      // requested by the user. Do not allow edits so we do not go over that limit any further. If
      // the journal rebuild failed, the journal writer will not be active, meaning we will not be
      // able to record the edit, causing file leaks. In both cases, we want to retry the clean up
      // so we can get out of this state!
      executor.execute(cleanupRunnable);
      return null;
    }

    // Flush the journal before creating files to prevent file leaks.
    journalWriter.writeUtf8(DIRTY).writeByte(' ').writeUtf8(key).writeByte('\n');
    journalWriter.flush();

    if (hasJournalErrors) {
      return null; // Don't edit; the journal can't be written.
    }

    if (entry == null) {
      entry = new Entry(key);
      lruEntries.put(key, entry);
    }
    Editor editor = new Editor(entry);
    entry.currentEditor = editor;
    return editor;
}
```

edit 方法返回对应 CacheEntry 的 editor 编辑器。



**总结** 

总结起来 DiskLruCache 主要有以下几个特点： 

- 通过 LinkedHashMap 实现 LRU 替换 
- 通过本地维护 Cache 操作日志保证 Cache 原子性与可用性，同时为防止日志过分膨胀定时执行日志精简 
- 每一个 Cache 项对应两个状态副本：DIRTY,CLEAN。CLEAN 表示当前可用状态 Cache，外部访问到的 cache 快照均为 CLEAN 状态；DIRTY 为更新态 Cache。由于更新和创建都只操作 DIRTY 状态副本，实现了 Cache 的读写分离 
- 每一个 Cache 项有四个文件，两个状态（DIRTY,CLEAN）,每个状态对应两个文件：一个文件存储 Cache meta 数据，一个文件存储 Cache 内容数据

#### 连接池

通过维护连接池，最大限度重用现有连接，减少网络连接的创建开销，以此提升网络请求效率。

##### keep-alive 机制

在 HTTP1.0 中 HTTP 的请求流程如下： 

![http1](https://img-blog.csdnimg.cn/20210512154755112.png)

这种方法的好处是简单，各个请求互不干扰。但在复杂的网络请求场景下这种方式几乎不可用。例如：浏览器加载一个 HTML 网页，HTML 中可能需要加载数十个资源，典型场景下这些资源中大部分来自同一个站点。按照 HTTP1.0 的做法，这需要建立数十个 TCP 连接，每个连接负责一个资源请求。创建一个 TCP 连接需要 3 次握手，而释放连接则需要 2 次或 4 次握手。重复的创建和释放连接极大地影响了网络效率，同时也增加了系统开销。为了有效地解决这一问题，HTTP/1.1 提出了 Keep-Alive 机制：当一个 HTTP 请求的数据传 输结束后，TCP 连接不立即释放，如果此时有新的 HTTP 请求，且其请求的 Host 通上次请求相同，则可以直接复用为释放的 TCP 连接，从而省去了 TCP 的释放和再次创建的开销，减少了网络延时: 

![](https://img-blog.csdnimg.cn/2021051215475552.png)

在现代浏览器中，一般同时开启 6～8 个 keepalive connections 的 socket 连接，并保持一 定的链路生命，当不需要时再关闭；而在服务器中，一般是由软件根据负载情况(比如 FD 最 

大值、Socket 内存、超时时间、栈内存、栈数量等)决定是否主动关闭。

##### HTTP/2 

在 HTTP/1.x 中，如果客户端想发起多个并行请求必须建立多个 TCP 连接，这无疑增大了网络开销。另外 HTTP/1.x 不会压缩请求和响应报头，导致了不必要的网络流量；HTTP/1.x 不支持资源优先级导致底层 TCP 连接利用率低下。而这些问题都是 HTTP/2 要着力解决的。简单来说 HTTP/2 主要解决了以下问题： 

- 报头压缩：HTTP/2 使用 HPACK 压缩格式压缩请求和响应报头数据，减少不必要流量开销 
- 请求与响应复用：HTTP/2 通过引入新的二进制分帧层实现了完整的请求和响应复用，客户端和服务器可以将 HTTP 消息分解为互不依赖的帧，然后交错发送，最后再在另一端将其重新组装 
- 指定数据流优先级：将 HTTP 消息分解为很多独立的帧之后，我们就可以复用多个数据流中的帧，客户端和服务器交错发送和传输这些帧的顺序就成为关键的性能决定因素。为了做到这一点，HTTP/2 标准允许每个数据流都有一个关联的权重和依赖关系 
- 流控制：HTTP/2 提供了一组简单的构建块，这些构建块允许客户端和服务器实现其自己的数据流和连接级流控制 

HTTP/2 所有性能增强的核心在于新的二进制分帧层，它定义了如何封装 HTTP 消息并在客户端与服务器之间进行传输: 

![](https://img-blog.csdnimg.cn/20210512155212866.png)

同时 HTTP/2 引入了三个新的概念： 

- 数据流：基于 TCP 连接之上的逻辑双向字节流，对应一个请求及其响应。客户端每发起一个请求就建立一个数据流，后续该请求及其响应的所有数据都通过该数据流传输 
- 消息：一个请求或响应对应的一系列数据帧
- 帧：HTTP/2 的最小数据切片单位 

上述概念之间的逻辑关系： 

- 所有通信都在一个 TCP 连接上完成，此连接可以承载任意数量的双向数据流 
- 每个数据流都有一个唯一的标识符和可选的优先级信息，用于承载双向消息 
- 每条消息都是一条逻辑 HTTP 消息（例如请求或响应），包含一个或多个帧 
- 帧是最小的通信单位，承载着特定类型的数据，例如 HTTP 标头、消息负载，等等。 来自不同数据流的帧可以交错发送，然后再根据每个帧头的数据流标识符重新组装 
- 每个 HTTP 消息被分解为多个独立的帧后可以交错发送，从而在宏观上实现了多个请求或响应并行传输的效果。这类似于多进程环境下的时间分片机制

![](https://img-blog.csdnimg.cn/20210512155522106.png)



无论是 HTTP/1.1 的 Keep-Alive 机制还是 HTTP/2 的多路复用机制，在实现上都需要引入连接池来维护网络连接。接下来看下 OkHttp 中的连接池实现。 

##### ConnectionPool

OkHttp 内部通过 ConnectionPool 来管理连接池，首先来看下 ConnectionPool 的主要成员：

```java
public final class ConnectionPool {
  /**
   * Background threads are used to cleanup expired connections. There will be at most a single
   * thread running per connection pool. The thread pool executor permits the pool itself to be
   * garbage collected.
   */
  private static final Executor executor = new ThreadPoolExecutor(0 /* corePoolSize */,
      Integer.MAX_VALUE /* maximumPoolSize */, 60L /* keepAliveTime */, TimeUnit.SECONDS,
      new SynchronousQueue<Runnable>(), Util.threadFactory("OkHttp ConnectionPool", true));

  /** The maximum number of idle connections for each address. */
  private final int maxIdleConnections;
  private final long keepAliveDurationNs;
  private final Runnable cleanupRunnable = new Runnable() {
    @Override public void run() {
      ...
    }
  };

  ...
  //返回符合要求的可重用连接，如果没有返回 null
  @Nullable RealConnection get(Address address, StreamAllocation streamAllocation, Route route) {
    assert (Thread.holdsLock(this));
    for (RealConnection connection : connections) {
      if (connection.isEligible(address, route)) {
        streamAllocation.acquire(connection, true);
        return connection;
      }
    }
    return null;
  }

  //去除重复连接。主要针对多路复用场景下一个 address 只需要一个连接
  @Nullable Socket deduplicate(Address address, StreamAllocation streamAllocation) {
    assert (Thread.holdsLock(this));
    for (RealConnection connection : connections) {
      if (connection.isEligible(address, null)
          && connection.isMultiplexed()
          && connection != streamAllocation.connection()) {
        return streamAllocation.releaseAndAcquire(connection);
      }
    }
    return null;
  }

  //将连接加入连接池
  void put(RealConnection connection) {
    assert (Thread.holdsLock(this));
    if (!cleanupRunning) {
      cleanupRunning = true;
      executor.execute(cleanupRunnable);
    }
    connections.add(connection);
  }

  //当有连接空闲时唤起 cleanup 线程清洗连接池
  boolean connectionBecameIdle(RealConnection connection) {
    assert (Thread.holdsLock(this));
    if (connection.noNewStreams || maxIdleConnections == 0) {
      connections.remove(connection);
      return true;
    } else {
      notifyAll(); // Awake the cleanup thread: we may have exceeded the idle connection limit.
      return false;
    }
  }

  ...
  //扫描连接池，清除空闲连接
  long cleanup(long now) {
    ...
  }
  ...
}

```

相关概念： 

- Call：对 Http 请求的封装 
- Connection/RealConnection:物理连接的封装，其内部有 List<WeakReference<StreamAllocation>>的引用计数 
- StreamAllocation: okhttp 中引入了 StreamAllocation 负责管理一个连接上的流，同时在 connection 中也通过一个 StreamAllocation 的引用的列表来管理一个连接的流，从而使得连接与流之间解耦。
- connections: Deque 双端队列，用于维护连接的容器
- routeDatabase:用来记录连接失败的 Route 的黑名单，当连接失败的时候就会把失败的线路加进去 

**实例化** 

首先来看下 ConnectionPool 的实例化过程，一个 OkHttpClient 只包含一个 ConnectionPool，其实例化过程也在 OkHttpClient 的实例化过程中实现，值得一提的是 ConnectionPool 各个方法的调用并没有直接对外暴露，而是通过 OkHttpClient 的 Internal 接口统一对外暴露：

```java
public class OkHttpClient implements Cloneable, Call.Factory {
  private static final List<Protocol> DEFAULT_PROTOCOLS = Util.immutableList(
      Protocol.HTTP_2, Protocol.SPDY_3, Protocol.HTTP_1_1);

  private static final List<ConnectionSpec> DEFAULT_CONNECTION_SPECS = Util.immutableList(
      ConnectionSpec.MODERN_TLS, ConnectionSpec.COMPATIBLE_TLS, ConnectionSpec.CLEARTEXT);

  static {
    Internal.instance = new Internal() {
      @Override public void addLenient(Headers.Builder builder, String line) {
        builder.addLenient(line);
      }

      @Override public void addLenient(Headers.Builder builder, String name, String value) {
        builder.addLenient(name, value);
      }

      @Override public void setCache(OkHttpClient.Builder builder, InternalCache internalCache) {
        builder.setInternalCache(internalCache);
      }

      @Override public InternalCache internalCache(OkHttpClient client) {
        return client.internalCache();
      }

      @Override public boolean connectionBecameIdle(
          ConnectionPool pool, RealConnection connection) {
        return pool.connectionBecameIdle(connection);
      }

      @Override public RealConnection get(
          ConnectionPool pool, Address address, StreamAllocation streamAllocation) {
        return pool.get(address, streamAllocation);
      }

      @Override public void put(ConnectionPool pool, RealConnection connection) {
        pool.put(connection);
      }

      @Override public RouteDatabase routeDatabase(ConnectionPool connectionPool) {
        return connectionPool.routeDatabase;
      }

      @Override
      public void callEnqueue(Call call, Callback responseCallback, boolean forWebSocket) {
        ((RealCall) call).enqueue(responseCallback, forWebSocket);
      }

      @Override public StreamAllocation callEngineGetStreamAllocation(Call call) {
        return ((RealCall) call).engine.streamAllocation;
      }

      @Override
      public void apply(ConnectionSpec tlsConfiguration, SSLSocket sslSocket, boolean isFallback) {
        tlsConfiguration.apply(sslSocket, isFallback);
      }

      @Override public HttpUrl getHttpUrlChecked(String url)
          throws MalformedURLException, UnknownHostException {
        return HttpUrl.getChecked(url);
      }
    };
}
```

Internal 的唯一实现在 OkHttpClient 中，OkHttpClient 通过这种方式暴露其 API 给外部类使用。

ConnectionPool 内部通过一个双端队列(dequeue)来维护当前所有连接，主要涉及到的操作 

包括： 

- put：放入新连接 
- get：从连接池中获取连接 
- evictAll：关闭所有连接
- connectionBecameIdle：连接变空闲后调用清理线程 
- deduplicate：清除重复的多路复用线程

**StreamAllocation.findConnection** 

get 是 ConnectionPool 中最为重要的方法，StreamAllocation 在其 findConnection 方法内 

部通过调用 get 方法为其找到 stream 找到合适的连接，如果没有则新建一个连接。首先来看下 findConnection 的逻辑：

```java
private RealConnection findConnection(int connectTimeout, int readTimeout, int writeTimeout,
      int pingIntervalMillis, boolean connectionRetryEnabled) throws IOException {
    boolean foundPooledConnection = false;
    RealConnection result = null;
    Route selectedRoute = null;
    Connection releasedConnection;
    Socket toClose;
    synchronized (connectionPool) {
      if (released) throw new IllegalStateException("released");
      if (codec != null) throw new IllegalStateException("codec != null");
      if (canceled) throw new IOException("Canceled");

      // Attempt to use an already-allocated connection. We need to be careful here because our
      // already-allocated connection may have been restricted from creating new streams.
      releasedConnection = this.connection;
      toClose = releaseIfNoNewStreams();
      if (this.connection != null) {
        // We had an already-allocated connection and it's good.
        result = this.connection;
        releasedConnection = null;
      }
      if (!reportedAcquired) {
        // If the connection was never reported acquired, don't report it as released!
        releasedConnection = null;
      }

      if (result == null) {
        // Attempt to get a connection from the pool.
        Internal.instance.get(connectionPool, address, this, null);
        if (connection != null) {
          foundPooledConnection = true;
          result = connection;
        } else {
          selectedRoute = route;
        }
      }
    }
    closeQuietly(toClose);

    if (releasedConnection != null) {
      eventListener.connectionReleased(call, releasedConnection);
    }
    if (foundPooledConnection) {
      eventListener.connectionAcquired(call, result);
    }
    if (result != null) {
      // If we found an already-allocated or pooled connection, we're done.
      return result;
    }

    // If we need a route selection, make one. This is a blocking operation.
    boolean newRouteSelection = false;
    if (selectedRoute == null && (routeSelection == null || !routeSelection.hasNext())) {
      newRouteSelection = true;
      routeSelection = routeSelector.next();
    }

    synchronized (connectionPool) {
      if (canceled) throw new IOException("Canceled");

      if (newRouteSelection) {
        // Now that we have a set of IP addresses, make another attempt at getting a connection from
        // the pool. This could match due to connection coalescing.
        List<Route> routes = routeSelection.getAll();
        for (int i = 0, size = routes.size(); i < size; i++) {
          Route route = routes.get(i);
          Internal.instance.get(connectionPool, address, this, route);
          if (connection != null) {
            foundPooledConnection = true;
            result = connection;
            this.route = route;
            break;
          }
        }
      }

      if (!foundPooledConnection) {
        if (selectedRoute == null) {
          selectedRoute = routeSelection.next();
        }

        // Create a connection and assign it to this allocation immediately. This makes it possible
        // for an asynchronous cancel() to interrupt the handshake we're about to do.
        route = selectedRoute;
        refusedStreamCount = 0;
        result = new RealConnection(connectionPool, selectedRoute);
        acquire(result, false);
      }
    }

    // If we found a pooled connection on the 2nd time around, we're done.
    if (foundPooledConnection) {
      eventListener.connectionAcquired(call, result);
      return result;
    }

    // Do TCP + TLS handshakes. This is a blocking operation.
    result.connect(connectTimeout, readTimeout, writeTimeout, pingIntervalMillis,
        connectionRetryEnabled, call, eventListener);
    routeDatabase().connected(result.route());

    Socket socket = null;
    synchronized (connectionPool) {
      reportedAcquired = true;

      // Pool the connection.
      Internal.instance.put(connectionPool, result);

      // If another multiplexed connection to the same address was created concurrently, then
      // release this connection and acquire that one.
      if (result.isMultiplexed()) {
        socket = Internal.instance.deduplicate(connectionPool, address, this);
        result = connection;
      }
    }
    closeQuietly(socket);

    eventListener.connectionAcquired(call, result);
    return result;
}
```

其主要逻辑大致分为以下几个步骤： 

- 查看当前 streamAllocation 是否有之前已经分配过的连接，有则直接使用 
- 从连接池中查找可复用的连接，有则返回该连接
- 配置路由，配置后再次从连接池中查找是否有可复用连接，有则直接返回 
- 新建一个连接，并修改其 StreamAllocation 标记计数，将其放入连接池中 
- 查看连接池是否有重复的多路复用连接，有则清除 

**ConnectionPool.get**

```java
RealConnection get(Address address, StreamAllocation streamAllocation, Route route) {
    assert (Thread.holdsLock(this));
    for (RealConnection connection : connections) {
      if (connection.isEligible(address, route)) {
        streamAllocation.acquire(connection, true);
        return connection;
      }
    }
    return null;
  }
```

其逻辑比较简单，遍历当前连接池，如果有符合条件的连接则修改器标记计数，然后返回。 

这里的关键逻辑在 RealConnection.isEligible 方法： 

```java
public boolean isEligible(Address address, @Nullable Route route) {
    // If this connection is not accepting new streams, we're done.
    if (allocations.size() >= allocationLimit || noNewStreams) return false;

    // If the non-host fields of the address don't overlap, we're done.
    if (!Internal.instance.equalsNonHost(this.route.address(), address)) return false;

    // If the host exactly matches, we're done: this connection can carry the address.
    if (address.url().host().equals(this.route().address().url().host())) {
      return true; // This connection is a perfect match.
    }

    // At this point we don't have a hostname match. But we still be able to carry the request if
    // our connection coalescing requirements are met. See also:
    // https://hpbn.co/optimizing-application-delivery/#eliminate-domain-sharding
    // https://daniel.haxx.se/blog/2016/08/18/http2-connection-coalescing/

    // 1. This connection must be HTTP/2.
    if (http2Connection == null) return false;

    // 2. The routes must share an IP address. This requires us to have a DNS address for both
    // hosts, which only happens after route planning. We can't coalesce connections that use a
    // proxy, since proxies don't tell us the origin server's IP address.
    if (route == null) return false;
    if (route.proxy().type() != Proxy.Type.DIRECT) return false;
    if (this.route.proxy().type() != Proxy.Type.DIRECT) return false;
    if (!this.route.socketAddress().equals(route.socketAddress())) return false;

    // 3. This connection's server certificate's must cover the new host.
    if (route.address().hostnameVerifier() != OkHostnameVerifier.INSTANCE) return false;
    if (!supportsUrl(address.url())) return false;

    // 4. Certificate pinning must match the host.
    try {
      address.certificatePinner().check(address.url().host(), handshake().peerCertificates());
    } catch (SSLPeerUnverifiedException e) {
      return false;
    }

    return true; // The caller's address can be carried by this connection.
}
```

- 连接没有达到共享上限 
- 非 host 域必须完全一样 
- 如果此时 host 域也相同，则符合条件，可以被复用 
- 如果 host 不相同，在 HTTP/2 的域名切片场景下一样可以复用

**deduplicate**

deduplicate 方法主要是针对在 HTTP/2 场景下多个多路复用连接清除的场景。如果当前连接是 HTTP/2，那么所有指向该站点的请求都应该基于同一个 TCP 连接：

```java
Socket deduplicate(Address address, StreamAllocation streamAllocation) {
    assert (Thread.holdsLock(this));
    for (RealConnection connection : connections) {
      if (connection.isEligible(address, null)
          && connection.isMultiplexed()
          && connection != streamAllocation.connection()) {
        return streamAllocation.releaseAndAcquire(connection);
      }
    }
    return null;
}
```

```java
long cleanup(long now) {
    int inUseConnectionCount = 0;
    int idleConnectionCount = 0;
    RealConnection longestIdleConnection = null;
    long longestIdleDurationNs = Long.MIN_VALUE;

    // Find either a connection to evict, or the time that the next eviction is due.
    synchronized (this) {
      for (Iterator<RealConnection> i = connections.iterator(); i.hasNext(); ) {
        RealConnection connection = i.next();

        // If the connection is in use, keep searching.
        if (pruneAndGetAllocationCount(connection, now) > 0) {
          inUseConnectionCount++;
          continue;
        }

        idleConnectionCount++;

        // If the connection is ready to be evicted, we're done.
        long idleDurationNs = now - connection.idleAtNanos;
        if (idleDurationNs > longestIdleDurationNs) {
          longestIdleDurationNs = idleDurationNs;
          longestIdleConnection = connection;
        }
      }

      if (longestIdleDurationNs >= this.keepAliveDurationNs
          || idleConnectionCount > this.maxIdleConnections) {
        // We've found a connection to evict. Remove it from the list, then close it below (outside
        // of the synchronized block).
        connections.remove(longestIdleConnection);
      } else if (idleConnectionCount > 0) {
        // A connection will be ready to evict soon.
        return keepAliveDurationNs - longestIdleDurationNs;
      } else if (inUseConnectionCount > 0) {
        // All connections are in use. It'll be at least the keep alive duration 'til we run again.
        return keepAliveDurationNs;
      } else {
        // No connections, idle or in use.
        cleanupRunning = false;
        return -1;
      }
    }

    closeQuietly(longestIdleConnection.socket());

    // Cleanup again immediately.
    return 0;
}
```

其基本逻辑如下： 

- 遍历连接池中所有连接，标记泄露连接 
- 如果被标记的连接满足(空闲 socket 连接超过 5 个&&keepalive 时间大于 5 分钟)，就将此连接从 Deque 中移除，并关闭连接，返回 0，也就是将要执行 wait(0)，提醒立刻再次扫描 
- 如果(目前还可以塞得下 5 个连接，但是有可能泄漏的连接(即空闲时间即将达到 5 分钟))，就返回此连接即将到期的剩余时间，供下次清理 
- 如果(全部都是活跃的连接)，就返回默认的 keep-alive 时间，也就是 5 分钟后再执行清理

pruneAndGetAllocationCount 负责标记并找到不活跃连接

```java
private int pruneAndGetAllocationCount(RealConnection connection, long now) {
    List<Reference<StreamAllocation>> references = connection.allocations;
    for (int i = 0; i < references.size(); ) {
      Reference<StreamAllocation> reference = references.get(i);

      if (reference.get() != null) {
        i++;
        continue;
      }

      // We've discovered a leaked allocation. This is an application bug.
      StreamAllocation.StreamAllocationReference streamAllocRef =
          (StreamAllocation.StreamAllocationReference) reference;
      String message = "A connection to " + connection.route().address().url()
          + " was leaked. Did you forget to close a response body?";
      Platform.get().logCloseableLeak(message, streamAllocRef.callStackTrace);

      references.remove(i);
      connection.noNewStreams = true;

      // If this was the last allocation, the connection is eligible for immediate eviction.
      if (references.isEmpty()) {
        connection.idleAtNanos = now - keepAliveDurationNs;
        return 0;
      }
    }

    return references.size();
}
```

OkHttp 的连接池通过计数+标记清理的机制来管理连接池，使得无用连接可以被会回收，并保持多个健康的 keep-alive 连接。这也是 OkHttp 的连接池能保持高效的关键原因。