# mlck

mlck is an experimental IRC gateway for Malíček.

## Operation

A virtual server in a sense, mlck authentication is forwarded to the configured
[Malíček endpoint](https://github.com/contyk/malicek), which in turn forwards
the request to [Alík](https://alik.cz/).  Once authenticated, all currently
active Alík users are visible as IRC users, with Alík rooms being presented as
channels.

The server periodically polls the endpoint for the current status of channels
the user has joined.  It also sends keepalive messages to connected channels to
prevent idling out.

For all operations, UTF-8 is assumed.

All users are identified as `<nick>!<id>@alik.cz`, where `id` is Alík's unique
string ID, as opposed to the numerical ones.

In general, the server aims to be fully compatible with [RFC
2812](https://tools.ietf.org/html/rfc2812#section-3.2.1), with some modern
extensions.  However, not all commands or options are fully supported as not
all of them make sense in the context of this workload.

## Configuration

Currently hardcoded.

## Supported commands

### `NICK`

Supported and required to register the connection.  Further nickname
changes are not allowed and return an error.

### `USER`

Somewhat supported and required to register the connection.  Arguments are
ignored as this is the only actual user of the virtual server.

Succesful connections return `RPL_WELCOME`, while unsuccessful connections
return `ERR_PASSWDMISMATCH`.

### `PASS`

Supported and required to register the connection.  Unauthenticated connections
are not accepted as they cannot be used to established a proper Malíček
session.

### `JOIN`

Fully supported, including non-standard forms.

Users can `JOIN` a single or a list of channels.  They can also join `0` to
`PART` all currently joined channels.  They can also `JOIN` with no arguments,
which does nothing.

Successful joins send `RPL_TOPIC` and `RPL_NAMEREPLY`.  Topic is the actual
room name.

### `PART`

Fully supported.

Users can leave both a single or a lsit of channels.

### `QUIT`

Somewhat supported.  `QUIT` results in the session being closed but no explicit
`PART` is currently being sent to the connected channels.  This will most
likely change later.

Upon `QUIT`, the server closes the connection.

The server currently doesn't respond with `ERROR`, even though it should.

### `PING`

Somewhat supported.  The server doesn't initiate any pings on its own but
responds to client pings.  Targetted pings are always processed by the virtual
server, so tha `target` argument and the response is effectively all lies.
Lies, I tell you!

### `LIST`

Fully supported.

Users can `LIST` all visible and accessible channels with `LIST`, or limit the
output with wildcards.  Inaccessible channels are not listed.

### `WHO`

Currently unsupported.

The server sends `RPL_NAMEREPLY` upon joining but doesn't understand `WHO`
queries.

### `WHOIS`

Unsupported.

### `MODE`

Unsupported.

### `TOPIC`

Currently unsupported.

The server sends `RPL_TOPIC` upon joining but doesn't understand `TOPIC`
queries.  `TOPIC` with arguments will likely never fully work as it's
unsupported by the backends.

Channel topics currently represent the actual room names.  Alík also supports
room topics, which are currently unsupported by Malíček.  One potential
solution to this is the topic being a concatenated string of the room name and
its topic, with `TOPIC` changing only the later, if the user is an operator.

### `NAMES`

Currently unsupported.

However, the server sends `RPL_NAMEREPLY` upon joining a channel.

### `KICK`

Currently unsupported.

### `KICKBAN`

Currently unsupported.


## Channels

Channel names correspond to their Alík table room IDs.  These consist of only
lowercase ASCII letters and hyphens.  The actual channel name is provided as a
topic.

### Channel modes

Currently unsupported.

## Users

Users are presented with their Alík nicknames.  Do note these may not comply
with common IRC practices, include various punctuation symbols at any position
and a subset of Unicode characters.  Some may even include spaces.

Personal, direct messages are currently unsupported, although they might map to
Alík's personal inbox at some point.

Alík allows spaces in nicknames for special, non-regular users.  These are
illegal in IRC and the spaces are therefore replaced with underscores in these
cases.

### User modes

Currently unsupported.

### Deployment

mlck requires Python 3, generally 3.4 or later is recommended.

Nothing besides the standard library is required.

mlck connects to a Malíček endpoint, which needs to be deployed first.

Only all is set up, just run it:

`./mlck`

### License

Petr Šabata <contyk@contyk.dev>, 2020

Licensed under MIT/X.  See `LICENSE` for details.
