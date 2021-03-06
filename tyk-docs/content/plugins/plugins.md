---
date: 2017-03-24T12:59:42Z
title: Plugins
menu: "main"
weight: 80
url: "/plugins"
---

Tyk supports the use of the following plugins to extend Tyk functionality:

*   [Rich Plugins](/docs/plugins/rich-plugins/)
*   [JavaScript Vitual Machine Middleware](/docs/plugins/javascript-middleware/) (JSVM Middleware)
*   [Authentication Plugins](/docs/plugins/auth-plugins/)
*   [Golang native plugins](/docs/plugins/golang-plugins/golang-plugins/)

> **Note**: Plugins are only available for Multi-Cloud and On-Premises installations.

## <a name="plugin-requirements"></a>Requirements
All plugins (except [Golang native plugins](/docs/plugins/golang-plugins/golang-plugins/)) require the following addition to be made to your `tyk.conf` file:

```{.copyWrapper}
"coprocess_options": {
  "enable_coprocess": true
},
"enable_bundle_downloader": true,
"bundle_base_url": "http://my-bundle-server.com/bundles/",
"public_key_path": "/path/to/my/pubkey",
```


`enable_coprocess` enables the rich plugins feature.

`enable_bundle_downloader` enables the bundle downloader.

`bundle_base_url` is a base URL that will be used to download the bundle.

`public_key_path` sets a public key, this is used for verifying signed bundles. You may omit this if unsigned bundles are used.


For a [gRPC](/docs/plugins/rich-plugins/grpc/) rich plugin a further `coprocess_grpc_server` parameter is required within `coprocess_options`:

```{.copyWrapper}
"coprocess_grpc_server": "tcp://127.0.0.1:5555"
```

## <a name="plugin-differences"></a>Differences between Rich Plugins and JSVM middleware
The JavaScript Virtual Machine provides pluggable middleware that can modify a request on the fly and are designed to augment a running Tyk process, are easy to implement and run inside the Tyk process in a sandboxed ECMAScript interpreter. This is good, but there are some drawbacks with the JSVM:

*   **Performance**: JSVM is performant, but is not easy to optimise and is dependent on the [otto interpreter](https://github.com/robertkrimen/otto) - this is not ideal. The JSVM also requires a copy of the interpreter object for each request to be made, which can increase memory footprint.
*   **Extensibility**: JSVM is a limited interpreter, although it can use some NPM modules, it isn't NodeJS so writing interoperable code (especially with other DBs) is difficult.
*   **TCP Access**: The JSVM has no socket access so working with DB drivers and directly with Redis is not possible.

Rich Plugins can provide replacements for existing middleware functions (as opposed to augmentation) and are designed to be full-blown, optimised, highly capable services. They enable a full customised architecture to be built that integrates with a user's infrastructure.

Rich Plugins bring about the following improvements:

*   **Performance**: Run on STDIN (unix pipes), which are extremely fast and run in their own memory space, and so can be optimised for performance way beyond what the JSVM could offer.
*   **Extensibility**: By allowing any language to be used so long as GRPC is supported, the extensibility of a CPH is completely open.
*   **TCP Access**: Because a plugin is a separate process, it can have it's own low-level TCP connections opens to databases and services.

There are some caveats to plugins:

*   They must run as a single process.
*   They can implement some or any of the API hooks.
*   The APIs used *must* be specified in an API definition and are not global across APIs.
*   They must manage API-specific cases in the same process, only one CoProcess will be managed by a Tyk Instance.