Source: https://www.youtube.com/watch?v=RVZX4CwKhGE

Speaker: Matt Klein, Lyft

SoA networking makes observability and debugging hard.
At Lyft, developers were scared to make service calls and started
shipping fat libraries again. In a pologlot environment, this causes
maintainence nightmares, problems like partial
implementations of stuff like retries, rate limiting etc.

There is a need for a common solution which solves
all these issues.

## Envoy
Mission statement: The network should be transpatent to applications.
When network and applications problems do occur it should be easy to
determine the source of the problem.

* Out of process archectecture/sidecar model: Envoy is collocated with application.
Application talks to local envoy, envoy does the network stuff and returns the
response back to application.

* L3/L4 filter architecture: Envoy is a byte oriented proxy. Can be used for
things other than HTTP (eg: MongoDB, redis, TCP rate limiter etc)

* L7 HTTP filter archectecture: Envoy can also do filtering on headers, body
or custom plugins.

* HTTP/2: Supports grpc

* Service discovery and active/passive health checking

* Advanced load balancing: Retry, timeouts, circuit breaking, rate limiting, shadowing,
outlier detection etc

* Best in class observability: stats, logging, tracing

* Edge proxy: routing and TLS. Can act as nginx replacement.