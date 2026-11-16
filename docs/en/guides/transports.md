# Transports

Every binding takes these settings as ORB options, on the command line or
in the argument list of `ORB_init` (Java: also as properties). The option
names are omniORB's. See [ORB options](../reference/orb-options.md) for the
full list.

## Addresses: where a server listens and what it publishes

```sh
server -ORBendPoint giop:tcp::2810                          # all addresses, port 2810
server -ORBendPoint giop:tcp:10.0.0.5:2810                  # one address
server -ORBendPoint giop:tcp::2810 -ORBendPointPublish giop:tcp:public.example.org:2810
```

The IOR carries the address the clients will use. Behind NAT, in a
container or with port forwarding, publish the address the clients can
reach with `-ORBendPointPublish`. Without `-ORBendPoint`, the server
listens on any free port of all addresses.

## TLS (SSLIOP) {#tls-ssliop}

IIOP over TLS, compatible with omniORB's SSL transport, made with rustls
(no OpenSSL).

```sh
server -ORBendPoint giop:ssl::2811 \
       -ORBsslCAFile ca.pem -ORBsslKeyFile server.pem
client -ORBsslCAFile ca.pem
```

| Option | Meaning |
| --- | --- |
| `-ORBendPoint giop:ssl:HOST:PORT` | listen with TLS. With no `giop:tcp` endpoint, the server does not listen in plain TCP at all |
| `-ORBsslCAFile PATH` | the CA that checks the other side (PEM). A client without it does not use TLS |
| `-ORBsslKeyFile PATH`, `-ORBsslCertFile PATH` | own key and certificate (PEM; the key file may hold both, as with omniORB) |
| `-ORBsslVerifyClient none\|optional\|required` | whether the server asks for a client certificate (default `none`) |
| `-ORBsslVerifyHost 0\|1` | whether the client checks the host name in the server's certificate (default 1) |

A client with a CA uses only the TLS address of a server that offers one,
and never falls back to plain TCP. CSIv2 is not supported.

## Unix domain sockets (`giop:unix`)

omniORB's transport between processes on the same host, on Linux and
macOS.

```sh
server -ORBendPoint giop:unix:/run/myapp/calc.sock
server -ORBendPoint giop:unix:                  # a new socket in /tmp/omni-USER/
server -ORBendPoint giop:unix:/run/myapp/       # a new socket in that directory
server -ORBendPoint giop:unix: -ORBendPoint giop:tcp::2810   # both
```

The IOR names the socket and the host it is on. A client on the same host
uses the socket before TCP or TLS, and falls back to the other addresses if
it cannot connect; clients on other hosts ignore it. The server removes the
socket file when it shuts down. Unlike omniORB, it refuses to start over a
file that is not a socket, or a socket another process listens on.

## Bidirectional GIOP

A client that cannot be reached from outside (behind NAT or a firewall) can
still receive callbacks over the connection it opened to the server. Turn
it on at both ends:

```sh
server -ORBacceptBiDirectionalGIOP 1
client -ORBofferBiDirectionalGIOP 1
```

and put the client's callback objects in a POA with the
`BidirectionalPolicy` `BOTH`:

```python
from rustorb import BiDirPolicy, CORBA

both = orb.create_policy(
    BiDirPolicy.BIDIRECTIONAL_POLICY_TYPE,
    CORBA.Any(BiDirPolicy._tc_BidirectionalPolicyValue, BiDirPolicy.BOTH),
)
poa = root_poa.create_POA("callbacks", root_poa._get_the_POAManager(), [both])
```

## Connections

The ORB keeps one connection per server address and shares it among all
calls. Idle connections close after a while (`-ORBoutConScanPeriod`,
`-ORBinConScanPeriod`, in seconds), and `-ORBmaxServerConnections` limits
how many a server accepts.

More: [ssliop.md](https://github.com/rustorb/rustorb/blob/main/docs/ssliop.md),
[unix-transport.md](https://github.com/rustorb/rustorb/blob/main/docs/unix-transport.md),
[bidir-giop.md](https://github.com/rustorb/rustorb/blob/main/docs/bidir-giop.md),
[connections.md](https://github.com/rustorb/rustorb/blob/main/docs/connections.md).
