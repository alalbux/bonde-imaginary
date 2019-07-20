# BONDE imaginary 

**[Fast](#benchmarks) HTTP [microservice](http://microservices.io/patterns/microservices.html)** written in Go **for high-level image processing** backed by [bimg](https://github.com/h2non/bimg) and [libvips](https://github.com/jcupitt/libvips). `imaginary` can be used as private or public HTTP service for massive image processing with first-class support for [Docker](#docker) & [Heroku](#heroku).
It's almost dependency-free and only uses [`net/http`](http://golang.org/pkg/net/http/) native package without additional abstractions for better [performance](#performance).

## Prerequisites

- [libvips](https://github.com/jcupitt/libvips) 8.3+ (8.5+ recommended)
- C compatible compiler such as gcc 4.6+ or clang 3.0+
- Go 1.10+

## Installation

```bash
go get -u github.com/h2non/imaginary
```

Also, be sure you have the latest version of `bimg`:
```bash
go get -u gopkg.in/h2non/bimg.v1
```

## Clients

- [node.js/io.js](https://github.com/h2non/node-imaginary)

Feel free to send a PR if you created a client for other language.


## Command-line usage

```
Usage:
  imaginary -p 80
  imaginary -cors
  imaginary -concurrency 10
  imaginary -path-prefix /api/v1
  imaginary -enable-url-source
  imaginary -disable-endpoints form,health,crop,rotate
  imaginary -enable-url-source -allowed-origins http://localhost,http://server.com,http://*.example.org
  imaginary -enable-url-source -enable-auth-forwarding
  imaginary -enable-url-source -authorization "Basic AwDJdL2DbwrD=="
  imaginary -enable-placeholder
  imaginary -enable-url-source -placeholder ./placeholder.jpg
  imaginary -enable-url-signature -url-signature-key 4f46feebafc4b5e988f131c4ff8b5997
  imaginary -enable-url-source -forward-headers X-Custom,X-Token
  imaginary -h | -help
  imaginary -v | -version

Options:
  -a <addr>                 Bind address [default: *]
  -p <port>                 Bind port [default: 8088]
  -h, -help                 Show help
  -v, -version              Show version
  -path-prefix <value>      Url path prefix to listen to [default: "/"]
  -cors                     Enable CORS support [default: false]
  -gzip                     Enable gzip compression (deprecated) [default: false]
  -disable-endpoints        Comma separated endpoints to disable. E.g: form,crop,rotate,health [default: ""]
  -key <key>                Define API key for authorization
  -mount <path>             Mount server local directory
  -http-cache-ttl <num>     The TTL in seconds. Adds caching headers to locally served files.
  -http-read-timeout <num>  HTTP read timeout in seconds [default: 30]
  -http-write-timeout <num> HTTP write timeout in seconds [default: 30]
  -enable-url-source        Enable remote HTTP URL image source processing (?url=http://..)
  -enable-placeholder       Enable image response placeholder to be used in case of error [default: false]
  -enable-auth-forwarding   Forwards X-Forward-Authorization or Authorization header to the image source server. -enable-url-source flag must be defined. Tip: secure your server from public access to prevent attack vectors
  -forward-headers          Forwards custom headers to the image source server. -enable-url-source flag must be defined.
  -enable-url-signature     Enable URL signature (URL-safe Base64-encoded HMAC digest) [default: false]
  -url-signature-key        The URL signature key (32 characters minimum)
  -allowed-origins <urls>   Restrict remote image source processing to certain origins (separated by commas). Note: Origins are validated against host *AND* path. 
  -max-allowed-size <bytes> Restrict maximum size of http image source (in bytes)
  -certfile <path>          TLS certificate file path
  -keyfile <path>           TLS private key file path
  -authorization <value>    Defines a constant Authorization header value passed to all the image source servers. -enable-url-source flag must be defined. This overwrites authorization headers forwarding behavior via X-Forward-Authorization
  -placeholder <path>       Image path to image custom placeholder to be used in case of error. Recommended minimum image size is: 1200x1200
  -concurrency <num>        Throttle concurrency limit per second [default: disabled]
  -burst <num>              Throttle burst max cache size [default: 100]
  -mrelease <num>           OS memory release interval in seconds [default: 30]
  -cpus <num>               Number of used cpu cores.
                            (default for current machine is 8 cores)
```

Start the server in a custom port:
```bash
imaginary -p 8080
```

Also, you can pass the port as environment variable:
```bash
PORT=8080 imaginary
```

Enable HTTP server throttle strategy (max 10 requests/second):
```
imaginary -p 8080 -concurrency 10
```

Enable remote URL image fetching (then you can do GET request passing the `url=http://server.com/image.jpg` query param):
```
imaginary -p 8080 -enable-url-source
```

Mount local directory (then you can do GET request passing the `file=image.jpg` query param):
```
imaginary -p 8080 -mount ~/images
```

Enable authorization header forwarding to image origin server. `X-Forward-Authorization` or `Authorization` (by priority) header value will be forwarded as `Authorization` header to the target origin server, if one of those headers are present in the incoming HTTP request.
Security tip: secure your server from public access to prevent attack vectors when enabling this option:
```
imaginary -p 8080 -enable-url-source -enable-auth-forwarding
```

Or alternatively you can manually define an constant Authorization header value that will be always sent when fetching images from remote image origins. If defined, `X-Forward-Authorization` or `Authorization` headers won't be forwarded, and therefore ignored, if present.
**Note**:
```
imaginary -p 8080 -enable-url-source -authorization "Bearer s3cr3t"
```

Send fixed caching headers in the response. The headers can be set in either "cache nothing" or "cache for N seconds". By specifying `0` imaginary will send the "don't cache" headers, otherwise it sends headers with a TTL. The following example informs the client to cache the result for 1 year:
```
imaginary -p 8080 -enable-url-source -http-cache-ttl 31556926
```

Enable placeholder image HTTP responses in case of server error/bad request.
The placeholder image will be dynamically and transparently resized matching the expected image `width`x`height` define in the HTTP request params.
Also, the placeholder image will be also transparently converted to the desired image type defined in the HTTP request params, so the API contract should be maintained as much better as possible.

This feature is particularly useful when using `imaginary` as public HTTP service consumed by Web clients.
In case of error, the appropriate HTTP status code will be used to reflect the error, and the error details will be exposed serialized as JSON in the `Error` response HTTP header, for further inspection and convenience for API clients.
```
imaginary -p 8080 -enable-placeholder -enable-url-source
```

You can optionally use a custom placeholder image.
Since the placeholder image should fit a variety of different sizes, it's recommended to use a large image, such as `1200`x`1200`.
Supported custom placeholder image types are: `JPEG`, `PNG` and `WEBP`.
```
imaginary -p 8080 -placeholder=placeholder.jpg -enable-url-source
```

Enable URL signature (URL-safe Base64-encoded HMAC digest).

This feature is particularly useful to protect against multiple image operations attacks and to verify the requester identity.
```
imaginary -p 8080 -enable-url-signature -url-signature-key 4f46feebafc4b5e988f131c4ff8b5997
```

It is recommanded to pass key as environment variables:
```
URL_SIGNATURE_KEY=4f46feebafc4b5e988f131c4ff8b5997 imaginary -p 8080 -enable-url-signature
```

Increase libvips threads concurrency (experimental):
```
VIPS_CONCURRENCY=10 imaginary -p 8080 -concurrency 10
```

Enable debug mode:
```
DEBUG=* imaginary -p 8080
```

Or filter debug output by package:
```
DEBUG=imaginary imaginary -p 8080
```
