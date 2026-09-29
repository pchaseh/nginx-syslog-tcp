# nginx-syslog-tcp

[![License: BSD 2-Clause](https://img.shields.io/badge/license-BSD%202--Clause-blue.svg)](LICENSE)

## Overview

A patch for NGINX that adds TCP as a transport for its built-in syslog
logging support. UDP remains the default, so existing configurations continue to
work unchanged.

## Rationale

While UDP has its advantage as a transport layer with lower overhead, it makes no guarantees about message delivery. Large messages may also be fragmented and subsequently dropped by intermediary networks. TCP is useful when reliable, ordered transport is more important than minimal overhead.

> [!NOTE]
> The provided changes only uphold the same guarantees that TCP offers. In the case of a connection being completely severed, it's the responsibility of the calling module to issue another attempt with the same log.
## Installation

In order to use this patch, NGINX must be compiled from source with it included.

Clone a supported NGINX release and this repository into the same directory:

```console
git clone --branch release-1.31.5 --depth 1 https://github.com/nginx/nginx.git
git clone --depth 1 https://github.com/pchaseh/nginx-syslog-tcp.git
```

Apply the patch from the root of the NGINX source tree:

```console
cd nginx
patch -p1 < ../nginx-syslog-tcp/nginx-syslog_tcp.patch
```

A successful application ends without failed hunks or `.rej` files. You can
then [configure and compile NGINX from source][nginx-build] as usual, using the
options appropriate for your installation.

To use another supported version, replace `release-1.31.6` with a release tag
from the compatibility table below.

## Compatibility

The following releases were checked against clean official NGINX source trees:

| NGINX version | Supported |
| --- | :---: |
| 1.23.4 | Yes |
| 1.24.0 | Yes |
| 1.26.3 | Yes |
| 1.28.3 | Yes |
| 1.29.0 | Yes |
| 1.30.4 | Yes |
| 1.31.5 | Yes |
| 1.31.6 | Yes |

Verification was performed on September 29, 2026, using GCC 15.3.1 on Linux.
Versions not listed may work, but have not been verified.

## Configuration

After compiling the patched source, add `transport=tcp` to a syslog target in
an `access_log` or `error_log` directive:

```nginx
access_log syslog:server=1.2.3.4:1234,tag=access,transport=tcp;
error_log syslog:server=1.2.3.4:1234,tag=error,transport=tcp error;
```

The `transport` parameter accepts `tcp` or `udp`. If it is omitted, NGINX uses
UDP to preserve its existing behavior:

```nginx
access_log syslog:server=1.2.3.4:1234,tag=access,transport=udp;
```

Over TCP, each message is prefixed with its length, as described for
octet-counting framing in [RFC 6587][rfc6587]. The receiver must accept this
framing, as rsyslog does by default and syslog-ng's `syslog()` source does.

All other syslog parameters continue to follow the [NGINX syslog
documentation][nginx-syslog].

## License

This project is available under the [BSD 2-Clause License](LICENSE).

[nginx-build]: https://github.com/nginx/nginx#building-from-source
[nginx-syslog]: https://nginx.org/en/docs/syslog.html
[rfc6587]: https://datatracker.ietf.org/doc/html/rfc6587#section-3.4.1
