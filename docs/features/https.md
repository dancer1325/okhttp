HTTPS
=====

* OkHttp -- attempts to -- balance
  * **Connectivity**
    * -- to -- as many hosts as possible
    * == advanced hosts / run the latest versions of [boringssl](https://boringssl.googlesource.com/boringssl/) + less out of date hosts / running older versions of [OpenSSL](https://www.openssl.org/)
  * **Security** of the connection
    * == verification of the remote webserver / certificates + privacy of data exchanged -- with -- strong ciphers

* requisites by OkHttp, to negotiate a connection to an HTTPS server
  * [TLS versions](https://square.github.io/okhttp/4.x/okhttp/okhttp3/-tls-version/)
    * client / wants to prioritize
      * connectivity -> include obsolete TLS versions
      * security -> include ONLY latest TLS
  * [cipher suites](https://square.github.io/okhttp/4.x/okhttp/okhttp3/-cipher-suite/) / offer
    * client / wants to prioritize
      * connectivity -> include weak-by-design cipher suites
      * security -> include ONLY strongest cipher suites

* [ConnectionSpec](https://square.github.io/okhttp/4.x/okhttp/okhttp3/-connection-spec/)
  * implement 
    * Specific security vs connectivity decisions
  * -- follow the -- model [Google Cloud Policies](https://cloud.google.com/load-balancing/docs/ssl-policies-concepts)
    * [historical changes | this policy](../security/tls_configuration_history.md) 
  * built-in, by OkHttp
    * `RESTRICTED_TLS`
      * == secure configuration / try to meet stricter compliance requirements
    * `MODERN_TLS`
      * == secure configuration / -- connects to -- modern HTTPS servers
      * default one
    * `COMPATIBLE_TLS`
      * == secure configuration / -- connects to -- secure–but not current–HTTPS servers
      * uses
        * as fallback
    * `CLEARTEXT`
      * == insecure configuration / -- used for -- `http://` URLs

    ```java
    OkHttpClient client = new OkHttpClient.Builder()
        .connectionSpecs(Arrays.asList(ConnectionSpec.MODERN_TLS, ConnectionSpec.COMPATIBLE_TLS))
        .build();
    ```

  * TLS versions & cipher suites / `ConnectionSpec` -- can change -- / each release
    * _Example1:_ OkHttp 2.2 dropped support -- for -- SSL 3.0
      * Reason: 🧠due to [POODLE](https://googleonlinesecurity.blogspot.ca/2014/10/this-poodle-bites-exploiting-ssl-30.html) attack 🧠	 
    * _Example2:_ OkHttp 2.3 dropped support -- for -- [RC4](https://en.wikipedia.org/wiki/RC4#Security)
    * recommendation
      * ⭐stay up-to-date with OkHttp version ⭐

  * custom `ConnectionSpec` / custom set of TLS versions & cipher suites
    * _Example:_ configuration -- limited to -- 3 highly-regarded cipher suites / requires Android 5.0+

    ```java
    ConnectionSpec spec = new ConnectionSpec.Builder(ConnectionSpec.MODERN_TLS)
        .tlsVersions(TlsVersion.TLS_1_2)
        .cipherSuites(
              CipherSuite.TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,
              CipherSuite.TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,
              CipherSuite.TLS_DHE_RSA_WITH_AES_128_GCM_SHA256)
        .build();
    
    OkHttpClient client = new OkHttpClient.Builder()
        .connectionSpecs(Collections.singletonList(spec))
        .build();
    ```

### Debugging TLS Handshake Failures

* TLS handshake
  * requires
    * clients & servers -- must share -- common
      * TLS version
      * cipher suite
  * depends on
    * JVM or Android version
    * OkHttp version
    * web server configuration
  * _Example:_ requirements NOT meet

    ```
    Caused by: javax.net.ssl.SSLProtocolException: SSL handshake aborted: ssl=0x7f2719a89e80:
        Failure in SSL library, usually a protocol error
            error:14077410:SSL routines:SSL23_GET_SERVER_HELLO:sslv3 alert handshake
            failure (external/openssl/ssl/s23_clnt.c:770 0x7f2728a53ea0:0x00000000)
        at com.android.org.conscrypt.NativeCrypto.SSL_do_handshake(Native Method)
    ```

* [Qualys SSL Labs][qualys]
  * allows
    * checking a web server's configuration
* [OkHttp's TLS configuration history](../security/tls_configuration_history.md)
* Applications / expected to be installed | older Android devices -- should consider adopting -- [Google Play Services’ ProviderInstaller][provider_installer]
  * Reason: 🧠increase
    * security for users
    * connectivity with web servers 🧠

### Certificate Pinning ([.kt][CertificatePinningKotlin], [.java][CertificatePinningJava])

* OkHttp, by default, trusts or assumes
  * certificate authorities | host platform ->
    * maximizes connectivity
    * -- subject to -- certificate authority attacks
      * _Example:_ [2011 DigiNotar attack](https://www.computerworld.com/article/2510951/cybercrime-hacking/hackers-spied-on-300-000-iranians-using-fake-google-certificate.html)
  * your HTTPS servers’ certificates -- are signed by a -- certificate authority
* [CertificatePinner](https://square.github.io/okhttp/4.x/okhttp/okhttp3/-certificate-pinner/)
  * increases security
    * -- via -- restricting certificates & certificate authorities / are trusted
  * limits your server team’s abilities / update their TLS certificates
  * recommendation
    * verify its use it with **your server’s TLS administrator!**

=== ":material-language-kotlin: Kotlin"
    ```kotlin
      private val client = OkHttpClient.Builder()
          .certificatePinner(
              CertificatePinner.Builder()
                  .add("publicobject.com", "sha256/afwiKY3RxoMmLkuRW1l7QsPZTJPwDS2pdDROQjXw8ig=")
                  .build())
          .build()

      fun run() {
        val request = Request.Builder()
            .url("https://publicobject.com/robots.txt")
            .build()

        client.newCall(request).execute().use { response ->
          if (!response.isSuccessful) throw IOException("Unexpected code $response")

          for (certificate in response.handshake!!.peerCertificates) {
            println(CertificatePinner.pin(certificate))
          }
        }
      }
    ```
=== ":material-language-java: Java"
    ```java
      private final OkHttpClient client = new OkHttpClient.Builder()
          .certificatePinner(
              new CertificatePinner.Builder()
                  .add("publicobject.com", "sha256/afwiKY3RxoMmLkuRW1l7QsPZTJPwDS2pdDROQjXw8ig=")
                  .build())
          .build();

      public void run() throws Exception {
        Request request = new Request.Builder()
            .url("https://publicobject.com/robots.txt")
            .build();

        try (Response response = client.newCall(request).execute()) {
          if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);

          for (Certificate certificate : response.handshake().peerCertificates()) {
            System.out.println(CertificatePinner.pin(certificate));
          }
        }
      }
    ```

### Customizing Trusted Certificates ([.kt][CustomTrustKotlin], [.java][CustomTrustJava])

* replace the host platform’s certificate authorities  -- with -- your own set 
* recommendation
  * verify its use it with **your server’s TLS administrator!**

=== ":material-language-kotlin: Kotlin"
    ```kotlin
      private val client: OkHttpClient

      init {
        val trustManager = trustManagerForCertificates(trustedCertificatesInputStream())
        val sslContext = SSLContext.getInstance("TLS")
        sslContext.init(null, arrayOf<TrustManager>(trustManager), null)
        val sslSocketFactory = sslContext.socketFactory

        client = OkHttpClient.Builder()
            .sslSocketFactory(sslSocketFactory, trustManager)
            .build()
      }

      fun run() {
        val request = Request.Builder()
            .url("https://publicobject.com/helloworld.txt")
            .build()

        client.newCall(request).execute().use { response ->
          if (!response.isSuccessful) throw IOException("Unexpected code $response")

          for ((name, value) in response.headers) {
            println("$name: $value")
          }

          println(response.body!!.string())
        }
      }

      /**
       * Returns an input stream containing one or more certificate PEM files. This implementation just
       * embeds the PEM files in Java strings; most applications will instead read this from a resource
       * file that gets bundled with the application.
       */
      private fun trustedCertificatesInputStream(): InputStream {
        ... // Full source omitted. See sample.
      }

      private fun trustManagerForCertificates(inputStream: InputStream): X509TrustManager {
        ... // Full source omitted. See sample.
      }
    ```
=== ":material-language-java: Java"
    ```java
      private final OkHttpClient client;

      public CustomTrust() {
        X509TrustManager trustManager;
        SSLSocketFactory sslSocketFactory;
        try {
          trustManager = trustManagerForCertificates(trustedCertificatesInputStream());
          SSLContext sslContext = SSLContext.getInstance("TLS");
          sslContext.init(null, new TrustManager[] { trustManager }, null);
          sslSocketFactory = sslContext.getSocketFactory();
        } catch (GeneralSecurityException e) {
          throw new RuntimeException(e);
        }

        client = new OkHttpClient.Builder()
            .sslSocketFactory(sslSocketFactory, trustManager)
            .build();
      }

      public void run() throws Exception {
        Request request = new Request.Builder()
            .url("https://publicobject.com/helloworld.txt")
            .build();

        Response response = client.newCall(request).execute();
        System.out.println(response.body().string());
      }

      private InputStream trustedCertificatesInputStream() {
        ... // Full source omitted. See sample.
      }

      public SSLContext sslContextForTrustedCertificates(InputStream in) {
        ... // Full source omitted. See sample.
      }
    ```

 [CustomTrustJava]: https://github.com/square/okhttp/blob/master/samples/guide/src/main/java/okhttp3/recipes/CustomTrust.java
 [CustomTrustKotlin]: https://github.com/square/okhttp/blob/master/samples/guide/src/main/java/okhttp3/recipes/kt/CustomTrust.kt
 [CertificatePinningJava]: https://github.com/square/okhttp/blob/master/samples/guide/src/main/java/okhttp3/recipes/CertificatePinning.java
 [CertificatePinningKotlin]: https://github.com/square/okhttp/blob/master/samples/guide/src/main/java/okhttp3/recipes/kt/CertificatePinning.kt
 [provider_installer]: https://developer.android.com/training/articles/security-gms-provider
 [qualys]: https://www.ssllabs.com/ssltest/
