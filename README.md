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

All users are identified as `<nick>!<id>@alik.cz`, where `id` is Alík's unique
string ID, as opposed to the numerical ones.

All connections must be registered using the standard `PASS`, `NICK` & `USER`
sequence.  `ERR_NOTREGISTERED` or `ERR_ALREADYREGISTERED` are returned if the
order is not respected.

In general, the server aims to be fully compatible with [RFC
2812](https://tools.ietf.org/html/rfc2812#section-3.2.1), with some modern
extensions.  However, not all commands or options are fully supported as not
all of them make sense in the context of this workload.

Malíček `system` messages are translated into IRC messages where applicable.
If it's not possible, they're sent to the channel or user, by the `*` IRC user.
Additionally, upon joining a channel, the `*` user sends the visible room
message backlog to the channel.  While it is theoretically possible to send
those as regular `PRIVMSG` messages, not all clients accept these before
receiving the `JOIN` confirmation.  Sending them after would be misleading and
the timestamps would get mangled.

## Configuration

Currently hardcoded.  Everything's hardcoded.  Really.

## Commands

In order as specified in RFC 2812.

### `PASS`

Supported and required to register the connection.  Unauthenticated connections
are not accepted as they cannot be used to established a proper Malíček
session.

Sending `PASS` when already registered results in the `ERR_ALREADYREGISTERED`
error.

### `NICK`

Supported and required to register the connection.  Further nickname
changes are not allowed and return the `ERR_ALREADYREGISTERED` error.

Along with `USER` triggers the connection registration.  These can be
sent in either order.  See below.

### `USER`

Somewhat supported and required to register the connection.  Arguments are
ignored as this is the only actual user of the virtual server.

Succesful connections return `RPL_WELCOME`, while unsuccessful connections
return `ERR_PASSWDMISMATCH`.  `USER` messages after a successful registration
return `ERR_ALREADYREGISTERED`.

### `OPER`

All server operator requests are denied.

### `MODE`

No support for channel or user modes; however, a limited set of modes could be
considered.  Work in progress.  See the `Client.handle_mode()` documentation.

Currently returns `ERR_UNKNOWNCOMMAND`.

### `SERVICE`

Service registration is rejected.

### `QUIT`

Somewhat supported.  `QUIT` results in the session being closed but no explicit
`PART` is currently being sent to the connected channels.  This will most
likely change later.

The server sends a proper `ERROR` message and closes the connection.

### `SQUIT`

Does not make sense in our context.  Pretends the user has `ERR_NOPRIVILEGES`.

### `JOIN`

Fully supported, including non-standard forms.

Users can `JOIN` a single or a list of channels.  They can also join `0` to
`PART` all currently joined channels.  They can also `JOIN` with no arguments,
which does nothing.

Successful joins send `RPL_TOPIC` and `RPL_NAMEREPLY`.  Topic is the actual
room name and its real topic, if set.

### `PART`

Fully supported.

Users can leave both a single or a lsit of channels.

### `TOPIC`

Limited to channels the user has joined.  Supports querying topics, which are
the actual room names, plus their real topics, if set.

Setting topics is currently unsupported.

### `NAMES`

Fully supported.  Users can query all chnnels or a subset.

### `LIST`

Fully supported.

Users can `LIST` all visible and accessible channels with `LIST`, or limit the
output with wildcards.  Inaccessible channels are not listed.

The topics are actual room names.

### `INVITE`

Unsupported.

### `KICK`

Currently unsupported.

### `PRIVMSG`

Supported for channels.  Kinda, sorta.  Users can send messages to channels
they have joined.  Messaging channels they haven't will not work.

No meaningful errors are currently returned.

Messaging users directly doesn't work either.  Alík has no concept of private
messages not bound to channels, so those messages appear as specially-formatted
public messages when received.  Sending such messages will be implemented via
specially-formatted public messages instead.  It currently doesn't work.  Don't
try.

True private messages could map to Alík's inbox but none of our layers supports
that.

### `NOTICE`

Supported.  Handled by `PRIVMSG` as it's essentially the same thing.

### `MOTD`

Unsupported.

### `LUSERS`

Unsupported.

### `VERSION`

Unsupported.

### `STATS`

Unsupported.

### `LINKS`

Unsupported.

### `TIME`

Unsupported.

### `CONNECT`

Unsupported.

### `TRACE`

Unsupported.

### `ADMIN`

Unsupported.

### `INFO`

Unsupported.

### `SERVLIST`

Unsupported.

### `SQUERY`

Unsupported.


### `WHO`

Fully supported, including wildcards.

With no arguments, `WHO` aggregates users on all visible channels and prints
them out, sorted, as members of the `*` channel.  Wildcards can limit the
output.

Perhaps unusually, wildcards are supported in channel masks, too, but the query
must begin with a `#`.  Querying all channels without user aggregation can
therefore be achieved with `WHO #*`.

### `WHOIS`

Unsupported.

### `WHOWAS`

Unsupported.

### `KILL`

Unsupported.

### `PING`

Somewhat supported.  The server doesn't initiate any pings on its own but
responds to client pings.  Targetted pings are always processed by the virtual
server, so tha `target` argument and the response is effectively all lies.
Lies, I tell you!

### `PONG`

Accepted but ignored.  The server doesn't initiate any pings so this should
never happen.

### `ERROR`

Accepted but ignored.  `ERROR` messages should be sent by servers only, not
clients.

### `AWAY`

Unsupported.

### `REHASH`

Unsupported.

### `DIE`

Unsupported.

### `RESTART`

Unsupported.

### `SUMMON`

Unsupported.

### `USERS`

Unsupported.

### `OPERWALL`

Unsupported.

### `USERHOST`

Unsupported.

### `ISON`

Unsupported.

## Channels

Channel names correspond to their Alík table room IDs.  These consist of only
lowercase ASCII letters and hyphens.  The actual channel name is provided as a
topic.

Channels that cannot be joined (unlisted or locked) are not displayed as their
ID is not known.

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

Once everything is set up, just run it:

`./mlck`

### License

Petr Šabata <contyk@contyk.dev>, 2020

Licensed under MIT/X.  See `LICENSE` for details.
