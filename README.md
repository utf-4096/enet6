# ENet6

This library is a fork of https://github.com/lsalzman/enet, an amazing library and protocol 

This is a fork of [ENet](https://github.com/lsalzman/enet), a reliable UDP networking library which still doesn't support IPv6 in 2023 (and [ENet's author seems to dislike contributions](https://github.com/lsalzman/enet/issues/78)).

So here's a ENet fork with full support for dual-stack IPv4 / IPv6.

I'm also aiming at improving ENet (for example by adding a way to detect timeout)

## Build

You can build this library using either [XMake](https://xmake.io) or [CMake](https://cmake.org/download/).

XMake has full support for all desktop and mobile platforms (plus a simpler syntax) so I would suggest to prefer it over CMake.

Build using xmake:

```
xmake
```

You can then install it using `sudo xmake install` (system-wide) or `xmake install -o dest_folder`

## Frequently asked questions:

### Is there a tutorial? / How to use it?

You can check the docs folder or read the official [ENet tutorial](http://enet.bespin.org/Tutorial.html).

Here are the key differences between the official library and this fork:

1. Header files are in the enet6 folder instead of the enet folder (`#include <enet6/enet.h>`)
2. `enet_host_create` now takes a `ENetAddressType` as its first parameter. I suggest to use `ENET_ADDRESS_TYPE_ANY` on a server host (which will make it a dual-stack socket, able to handle IPv4 and IPv6). On a client host use either `ENET_ADDRESS_TYPE_IPV4` or `ENET_ADDRESS_TYPE_IPV6` depending on the ENetAddress you wish to connect to.
3. `ENetAddress` now has a `type` field, and its `host` field shouldn't be manipulated directly.
4. Peers disconnected by timing out now emit a `ENET_EVENT_TYPE_DISCONNECT_TIMEOUT` instead of a `ENET_EVENT_TYPE_DISCONNECT`

Examples:

#### Creating an ENet6 server**

```c
ENetAddress address;
ENetHost* server;
/* Bind the server to the default localhost.     */
/* A specific host address can be specified by   */
/* enet_address_set_host (& address, type, "x.x.x.x"); */

/* use enet_address_build_any to configure a ENetAddress */
enet_address_build_any(ENET_ADDRESS_TYPE_IPV6, &address);
/* Default port set by enet_address_build_any is ENET_PORT_ANY which means a random port, this is not convenient for a server */
/* Bind the server to port 1234. */
address.port = 1234;
server = enet_host_create (ENET_ADDRESS_TYPE_ANY, /* either has to match address->type or be ENET_ADDRESS_TYPE_ANY to dual stack the socket */
                           & address /* the address to bind the server host to */, 
                           32      /* allow up to 32 clients and/or outgoing connections */,
                           2      /* allow up to 2 channels to be used, 0 and 1 */,
                           0      /* assume any amount of incoming bandwidth */,
                           0      /* assume any amount of outgoing bandwidth */);
if (server == NULL)
{
    fprintf (stderr, 
             "An error occurred while trying to create an ENet server host.\n");
    exit (EXIT_FAILURE);
}
...
...
...
enet_host_destroy(server);
```

#### Connecting to a ENet6 client (using IPv4 or IPv6 depending on what's supported)

```c
ENetAddress address;
ENetEvent event;
ENetPeer* peer;
ENetHost* client;
/* Connect to some.server.net:1234. */
/* enet_address_set_host will resolve the target address using the target family */
/* if ENET_ADDRESS_TYPE_ANY is specified it will try to resolve using IPv6 if possible */
/* but may fallback on IPv4 */
enet_address_set_host (& address, ENET_ADDRESS_TYPE_ANY, "some.server.net");
address.port = 1234;
/* Initiate the connection, allocating the two channels 0 and 1. */

/* Create the enet host after resolving the address (allowing to create a IPv6 or IPv4 host) */
/* Using ENET_ADDRESS_TYPE_ANY type could also work but may fail on computers not supporting IPv6 at all */
client = enet_host_create (address.type,
                           NULL /* create a client host */,
                           1 /* only allow 1 outgoing connection */,
                           2 /* allow up 2 channels to be used, 0 and 1 */,
                           0 /* assume any amount of incoming bandwidth */,
                           0 /* assume any amount of outgoing bandwidth */);
if (client == NULL)
{
    fprintf (stderr, 
             "An error occurred while trying to create an ENet client host.\n");
    exit (EXIT_FAILURE);
}

peer = enet_host_connect (client, & address, 2, 0);    
if (peer == NULL)
{
   fprintf (stderr, 
            "No available peers for initiating an ENet connection.\n");
   exit (EXIT_FAILURE);
}
/* Wait up to 5 seconds for the connection attempt to succeed. */
if (enet_host_service (client, & event, 5000) > 0 &&
    event.type == ENET_EVENT_TYPE_CONNECT)
{
    puts ("Connection to some.server.net:1234 succeeded.");
    ...
    ...
    ...
}
else
{
    /* Either the 5 seconds are up or a disconnect event was */
    /* received. Reset the peer in the event the 5 seconds   */
    /* had run out without any significant event.            */
    enet_peer_reset (peer);
    puts ("Connection to some.server.net:1234 failed.");
}
...
...
...
enet_host_destroy(client);
```

### Is it compatible with enet?

Yes! You can connect to a regular enet server using a enet host created with the ENET_ADDRESS_TYPE_IPV4 address type (or ENET_ADDRESS_TYPE_ANY by using a IPv4-mapped IPv6). Regular enet clients can also connect to a enet6 host, if it was created using a ENET_ADDRESS_TYPE_ANY address type.

### But why?

ENet received **four** pull requests to add IPv6 support between 2013 and 2019, and no one has ever been accepted or dismissed by ENet author which seems to just ignore the problem and keep ENet as a growing old library with no support for IPv4.

As a network programming teacher, I'm constantly running into issues like ENet only supporting IPv4 and other forks (like [ENet-CSharp](https://github.com/nxrighthere/ENet-CSharp)) solely relying on IPv6 which may not work on some network/computers.

This implementation aims to be an updated version of ENet, with IPv6 support and some other stuff I can use to teach network programming.

Feel free to use it.

### Is there a closer version to libenet with only IPv6 support?

Yes! Check the ipv6 branch which was used for my IPv6 pull request to ENet

### Why bump from ENet 1.x.y to ENet6 6.x.y?

ENet has been in version 1.x.y for 15+ years, I'm catching up (jk).