OkHttp
======

* check [project website][okhttp]
* HTTP 
  * == modern applications network 
    * uses
      * exchange data & media
    * if you do efficiently -> your stuff
      * load faster
      * saves bandwidth

* OkHttp
  * == HTTP client / by default
    * HTTP/2 support / ALL requests -- can flow to the -- SAME host / share a socket
    * Connection pooling
      * if HTTP/2 is NOT available -> reduces request latency
    * Transparent GZIP
      * -- shrinks -- download sizes
    * Response caching
      * -- avoids, through the network -- repeated requests
  * if network has got problems -> OkHttp perseveres
    * _Example:_ try to recover the connection 
    * if your service has multiple IP addresses & somme connections fail -> OkHttp will attempt alternate addresses
      * Reason: 🧠 necessary for IPv4+IPv6 & services / hosted | redundant data centers
  * supports
    * modern TLS features (TLS 1.3, ALPN, certificate pinning)
      * OkHttp `3.12.x` -- lacks support for -- TLS 1.2
      * uses
        * fall back -- for -- broad connectivity
      * [updated to the dynamic TLS ecosystem][tls_history]
    * both
      * synchronous blocking calls
      * async calls with callbacks 
    * [Conscrypt][conscrypt] | Java platforms
      * integrates [BoringSSL](https://github.com/google/boringssl) -- with -- Java
      * if it's the first security provider -> used by OkHttp
    
        ```java
        Security.insertProviderAt(Conscrypt.newProvider(), 1);
        ```
  * request/response API -- is designed with --
    * fluent builders
    * immutability


Get a URL
---------

* [Full source][get_example]
  * goal
    * downloads a URL
    * prints its contents -- as a -- string 

    ```java
    OkHttpClient client = new OkHttpClient();
    
    String run(String url) throws IOException {
      Request request = new Request.Builder()
          .url(url)
          .build();
    
      try (Response response = client.newCall(request).execute()) {
        return response.body().string();
      }
    }
    ```


Post to a Server
----------------

* [Full source][post_example]
  * goal
    * posts data -- to a -- service


```java
public static final MediaType JSON = MediaType.get("application/json");

OkHttpClient client = new OkHttpClient();

String post(String url, String json) throws IOException {
  RequestBody body = RequestBody.create(json, JSON);
  Request request = new Request.Builder()
      .url(url)
      .post(body)
      .build();
  try (Response response = client.newCall(request).execute()) {
    return response.body().string();
  }
}
```

* [OkHttp Recipes page][recipes]
  * more examples    


Requirements
------------

* Android
  * OkHttp latest release -> Android 5.0+ (API level 21+)
  * OkHttp `3.12.x` branch -> Android 2.3+ (API level 9+)
* Java
  * OkHttp latest release -> Java 8+
  * OkHttp `3.12.x` branch -> Java 7+
* [Okio][okio] -- for --
  * high-performance I/O
  * [Kotlin standard library][kotlin]
* your platform's built-in TLS implementation
  * used by OkHttp


Recommendations
------------

* keep OkHttp up-to-date

Releases
--------

* [change log][changelog]
* latest release | [Maven Central](https://search.maven.org/artifact/com.squareup.okhttp3/okhttp/4.12.0/jar)

    ```kotlin
    implementation("com.squareup.okhttp3:okhttp:4.12.0")
    ```

* Snapshot builds | [available][snap]
* [R8 and ProGuard][r8_proguard] rules
* [bill of materials (BOM)][bom]
  * allows
    * keeping OkHttp artifacts | up to date & version compatibility

        ```kotlin
            dependencies {
               // define a BOM and its version
               implementation(platform("com.squareup.okhttp3:okhttp-bom:4.12.0"))
    
               // define any required OkHttp artifacts without version
               implementation("com.squareup.okhttp3:okhttp")
               implementation("com.squareup.okhttp3:logging-interceptor")
            }
        ```

MockWebServer
-------------

* included library
  * allows testing
    * HTTP
    * HTTPS
    * HTTP/2 clients
* latest release | [Maven Central](https://search.maven.org/artifact/com.squareup.okhttp3/mockwebserver/4.12.0/jar)

    ```kotlin
    testImplementation("com.squareup.okhttp3:mockwebserver:4.12.0")
    ```

GraalVM Native Image
--------------------

* [GraalVM](https://www.graalvm.org/)
* versions
  * NOT currently exist a final released version
  * use `5.0.0-alpha.2`
* _Example:_

    ```shell
    $ ./gradlew okcurl:nativeImage
    $ ./okcurl/build/graal/okcurl https://httpbin.org/get
    ```

License
-------

```
Copyright 2019 Square, Inc.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```

 [bom]: https://docs.gradle.org/6.2/userguide/platforms.html#sub:bom_import
 [changelog]: https://square.github.io/okhttp/changelog/
 [conscrypt]: https://github.com/google/conscrypt/
 [get_example]: https://raw.github.com/square/okhttp/master/samples/guide/src/main/java/okhttp3/guide/GetExample.java
 [kotlin]: https://kotlinlang.org/
 [okhttp3_pro]: https://raw.githubusercontent.com/square/okhttp/master/okhttp/src/main/resources/META-INF/proguard/okhttp3.pro
 [okhttp_312x]: https://github.com/square/okhttp/tree/okhttp_3.12.x
 [okhttp]: https://square.github.io/okhttp/
 [okio]: https://github.com/square/okio
 [post_example]: https://raw.github.com/square/okhttp/master/samples/guide/src/main/java/okhttp3/guide/PostExample.java
 [r8_proguard]: https://square.github.io/okhttp/features/r8_proguard/
 [recipes]: https://square.github.io/okhttp/recipes/
 [snap]: https://s01.oss.sonatype.org/content/repositories/snapshots/
 [tls_history]: https://square.github.io/okhttp/tls_configuration_history/
