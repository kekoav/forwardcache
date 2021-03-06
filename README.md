# forwardcache

[![Build Status](https://travis-ci.org/mikegleasonjr/forwardcache.svg?branch=master)](https://travis-ci.org/mikegleasonjr/forwardcache)
[![Coverage Status](https://codecov.io/gh/mikegleasonjr/forwardcache/branch/master/graph/badge.svg)](https://codecov.io/gh/mikegleasonjr/forwardcache)
[![GitHub license](https://img.shields.io/badge/license-Apache%202-blue.svg)](https://raw.githubusercontent.com/mikegleasonjr/forwardcache/master/LICENSE)
[![GoDoc](http://godoc.org/github.com/mikegleasonjr/forwardcache?status.svg)](http://godoc.org/github.com/mikegleasonjr/forwardcache)

A distributed forward caching proxy for Go's http.Client using [httpcache][httpcache] and heavily inspired by [groupcache][groupcache]. Backed by a lot of existing cache [backends][backends] thanks to httpcache. A per host LRU algorithm is provided to optionally front any existing cache. Like groupcache, forwardcache "is a client library as well as a server. It connects to its own peers."

In simple terms, it is a distributed cache for HEAD and GET requests. It follows the HTTP RFC so it will only cache cacheable responses (like browsers do).

Docs on [godoc.org][godoc]

<pre>
+---------------------------------+
|                                 |
|         origin servers          |
|                                 |
+--+-------+-------+-----------+--+
   ^       ^       ^           ^
   |       |       |           |
   |       |       |           |
+-----+ +-----+ +-----+     +-----+
|cache| |cache| |cache|     |cache|
+--+--+ +--+--+ +--+--+     +--+--+
|     | |     | |     |     |     |
|  1  +-+  2  +-+  3  +--|--+  N  |
|     | |     | |     |     |     |
+-----+ +-----+ +-----+     +-----+
</pre>

## Requirements

* Go 1.7 (using request's context)

## Motivation

* Needed requests to be cached
* Needed bandwidth to be spread among N peers
* Needed to cache HTTPS requests
* Needed a lightweight infrastructure

So I couldn't go with existing solutions like Apache Traffic Server or Squid.

## Process

When making a request through the pool...

1. the requested url is hashed to determine which peer is responisble to proxy the request to the origin
2. the peer is called, if the request is cached and valid, it is returned without contacting the origin
3. if the request is not cached, the request is fetched from the origin and cached if cacheable before being returned back to the client

## Example

```go
peer := NewPeer("http://10.0.1.1:3000")
peer.SetPool("http://10.0.1.1:3000", "http://10.0.1.2:3000")

// -or-

client := NewClient(WithPool("http://10.0.1.1:3000", "http://10.0.1.2:3000"))
peer = NewPeer("http://10.0.1.1:3000", WithClient(client))

// -then-

http.DefaultTransport = peer
http.Get("https://...js/1.5.7/angular.min.js")

// -or-

http.DefaultClient = peer.HTTPClient()
http.Get("https://...js/1.5.7/angular.min.js")

// -or-

c := peer.HTTPClient()
c.Get("https://...js/1.5.7/angular.min.js")

// ...

http.ListenAndServe(":3000", peer.Handler())
```

## Licence

Apache 2.0 (see LICENSE and NOTICE)






[httpcache]: https://github.com/gregjones/httpcache  "gregjones/httpcache"
[backends]: https://github.com/gregjones/httpcache#cache-backends  "cache backends"
[groupcache]: https://github.com/gregjones/httpcache#cache-backends  "golang/groupcache"
[singleflight]: https://godoc.org/golang.org/x/sync/singleflight "x/sync/singleflight"
[godoc]: http://godoc.org/github.com/mikegleasonjr/forwardcache "mikegleasonjr/forwardcache"
