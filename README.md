# System Essentials for WebAsssembly

## Introduction

WebAssembly is sandboxed by design and does not have access to functionality other than what is supported by its instruction set or provided to it via imports. So typically, if there is functionality the host supports but WebAssembly doesn't, one would provide an appropriate import. On the Web, one would import a Web API, while off the Web, one would utilize an import namespace like WASI.

However, some system features like obtaining the current time are essential to run basic (portable) WebAssembly modules both on and off the Web, and being able to access these features in a uniform way would help to battle fragmentation on an essential level that would otherwise require recompiling for different import namespaces or utilizing polyfills, which is generally good to avoid.

Hence, this document explores a new set of essential `system.*` instructions, also addressing related concerns raised in discussions like [[1](https://github.com/WebAssembly/WASI/issues/401)] and [[2](https://github.com/WebAssembly/design/issues/1407)].

## Motivation

Code reuse on and off the Web is desirable for the long-term success of the broader WebAssembly ecosystem, and even though different environments may impose different requirements on system APIs, some system APIs are essential and already common among the Web and Non-Web, so providing these in a portable way makes individual components reusable across ecosystems:

<p align="center">
  <img src="./code-reuse.svg" alt="Code reuse" />
</p>

In the diagram above, the example image encoder and cryptographic library only depend on core WebAssembly with System Essentials, so can be reused across ecosystems, in turn improving not only library consumer ergonomics, but also providing essential building blocks compiler and standard library authors can rely on.

## Instructions

* `system.time_local` obtains the current local time in milliseconds since Unix epoch.
  * `system.time_local : [] -> [i64]`

* `system.time_utc` obtains the current universal time in milliseconds since Unix epoch.
  * `system.time_utc : [] -> [i64]`

* `system.timezoneoffset` obtains the timezone offset between local and universal time in minutes.
  * `system.timezoneoffset : [] -> [i32]`

* `system.hrtime` obtains the system's monotonic high resolution time in nanoseconds since an arbitrary time in the past.
  * `system.hrtime : [] -> [i64]`

* `system.random` obtains cryptographically secure random bytes and stores them to memory.
  * `system.random : [i32, i32] -> []`

## Potential extensions

The following extensions are not feasible yet because they depend on future features such as Interface Types or may require further evaluation, but may become useful eventually:

* `system.log` logs a message to the system console.
  * `system.log : [string] -> []`

* Access to the system's IANA timezone database.

## Considerations

The list of instructions is not exhaustive and does not imply that instructions not yet mentioned aren't desirable. However, all `system.*` instructions must be safe, as in not negatively affecting the host, other applications or their data.

## Use cases

* Programs or libraries not specifically requiring Web or WASI APIs would not need additional non-portable imports in general, but are reusable by design.
  * Time(ing)-aware utility
  * Cryptographic utility
  * Debugging by printing to console
  * ...
* Compilers and standard libraries could replace custom solutions with System Essentials / would not have to choose between Web and WASI APIs.
* WASI applications could utilize System Essentials, reducing non-portable API surface, while also reducing polyfill overhead on the Web. For example, may only need to polyfill a file system.
* Web applications could utilize System Essentials, reducing the need for separate glue code. For example, `system.timezoneoffset` cannot trivially be imported (requires `new`ing a `Date` object).
