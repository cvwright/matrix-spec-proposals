# MSC2545: Emotes

Emotes.....emotes!

# Proposal
Emotes have at least a shortcode and an mxc uri. They are sent as `<img>` tags currently already in
the spec, as such existing clients *should* be able to render them (support for this is sadly poor,
even within riot flavours). To allow clients to distinguish emotes from other inline images, a new
property `data-mx-emote` is introduced. A client may chose to ignore the size attributes of emotes
when rendering, and instead pick the size based on other circumstances. This could e.g. be used to
display emotes in messages with only emotes and emoji larger than usual, which is commonly found in
messengers. Such an `<img>` tag of a shortcode `:emote:` and an mxc uri `mxc://example.org/emote`
could look as follows:

```html
<img data-mx-emote src="mxc://example.org/emote" alt=":emote:" title=":emote:" height="32" />
```

Both the `alt` and the `title` attributes are specified as they serve different purposes: `alt` is
displayed if e.g. the emote does not load. `title` is displayed e.g. on mouse hover over the emote.
The height is just a height that looks good on most devices with the normal, default font size.
No width is displayed as to not weirdly squish non-square emotes.

## Emote sources
In order to be able to send emotes the client needs to have a list of shortcodes and their corresponding
mxc uris. All the emote sources described here are merely suggestions on where to get emotes from.
A client is free to implement / design their own way of fetching emotes, e.g. via dropping image files
into a folder and uploading them on-the-fly. For cross-compatibility between clients it'd still be a
good idea to implement the emote sources described here. For this there are two different emote sources:

### User emotes
User emotes are per-user emotes that are defined in the users respective account data. The type for that
is `im.ponies.user_emotes` (later: `m.emotes`). The content is as following:

```json
{
  "short": {
    ":emote:": "mxc://example.org/blah",
    ":other-emote:": "mxc://example.org/other-blah"
  }
}
```

The emotes are defined inside of a dict called `short`, which stands for shortcode. Other, additional
keys may exist to define more metadata for the emotes. No such guide exists yet.

### Room emotes
Room emotes are per-room emotes that every user of a specific room can use inside of that room. They
are set with a state event of type `im.ponies.room_emotes` (later: `m.emotes`). The state key denotes a possible
pack, whereas the default one would be a blank state key. E.g. a discord bridge could set as state key
`de.sorunome.mx-puppet-bridge.discord` and have all the bridged emotes in said state event, keeping
bridged emotes from matrix emotes separate.

The content extends that of the user emotes: It uses the `short` key, which is a map of the shortcode
of the emote to its mxc uri. Additionally, an optional `pack` key can be set, which defines meta-information
on the pack. The following keys for `pack` are valid:

 - `displayname`: An easy displayname for the pack. Defaults to the room name, if it doesn't exist
 - `avatar_url`: The mxc uri of an avatar/icon to display for the pack. Defaults to the room name,
   if it doesn't exist.
 - `name`: A short identifier of the pack. Defaults to the normalized state key, and if the state
   key is blank it defaults to "room".
   
   Normalized means here, converting spaces to `-`, taking only alphanumerical characters, `-` and `_`,
   and casting it all to lowercase. In regex, this would be `[a-z0-9-_]+`.

As such, a `im.ponies.room_emotes` (later: `m.emotes`) state event could look like the following:

```json
{
  "short": {
    ":emote:": "mxc://example.org/blah",
    ":other-emote:": "mxc://example.org/other-blah"
  },
  "pack": {
    "displayname": "Emotes from Discord!",
    "avatar_url": "mxc://example.org/discord_guild",
    "name": "some_discord_guild"
  }
}
```

### Emote rooms
While room emotes are specific to a room and are only accessible within that room, emote rooms should
be accessible from everywhere. They do not differentiate themselves from room emotes at all, instead you
set an event in your account data of type `im.ponies.emote_rooms` (later: `m.emotes.rooms`) which outlines
which room emote states should be globally accessible for that user. For that, a `room` key contains
a map of room ids that map to state keys that map to an optional pack definition override.
If you are currently viewing a room that is in your `im.ponies.emote_rooms` (later: `m.emotes.rooms`)
it is expected that the client de-duplicates the packs, to only give suggestions for it onces.
The the contents of `im.ponies.emote_rooms` (later: `m.emotes.rooms`) could look like the following:

```json
{
  "rooms": {
    "!someroom:example.org": {
      "": {},
      "de.sorunome.mx-puppet-bridge.discord": {
        "displayname": "Overriden name",
        "short": "new_short_name"
      }
    },
    "!someotherroom:example.org": {
      "": {}
    }
  }
}
```

Here three emote packs are globally accessible to the user: Two defined in `!someroom:example.org`
(one with blank state key and one with state key `de.sorunome.mx-puppet-bridge.discord`) and one in
`!someotherroom:example.org`.

## Sending
In places where fancy tab-complete with the emote itself is not possible it is suggested that sending
the shortcode will convert to the img tag, e.g. sending `Hey there :wave:` will convert to `Hey there <img-for-wave>`.

If there are collisions due to the same emote shortcode being present as both room emote and user emote
a user could specify the emote source by writing e.g. `:room~wave:` or `:user~wave:`. Here the short
pack name is used for room emotes, and "user" for user emotes. If a room pack does not have a short
pack name and has a blank state key, then "room" is used.

# Ideas
 - ???

# Current implementations
## Emote rendering (rendering of the `<img>` tag)
 - riot-web
 - revolution
 - nheko
 - fluffychat
## Emote sending, using the mentioned events here
 - revolution
 - fluffychat

# Security Considerations
When sending an `<img>` tag in an end-to-end encrypted room, the client will make a request to fetch
said image, in this case an emote. As there is no way to encrypt content behind `<img>` tags yet,
this could potentially leak part of the conversation. This is **not** a new security consideration,
it already exists. This, however, isn't much different of posting someone a link in an e2ee chat and
the recipient opens the link. Additionally, images, and thus emotes, are often cached by the client,
not even necessarily leading to an http query.

# Alternatives
One can easily think of near infinite ways to implement emotes. The aspect of sending an `<img>` tag
and marking it as an emote with `data-mx-emote` seems to be pretty universal, though, so the question
is mainly on different emote sources. For that, a separate MSC, [MSC1951](https://github.com/matrix-org/matrix-doc/pull/1951)
already exists, so it is reasonable to compare this one with that one:

## Comparison with MSC1951
MSC1951 defines as only emote source a dedicated room. This MSC, however, also allows you to bind emotes
to your own account, offering greater flexibility. In MSC1951 there can also only be a single emote
pack in a room. This could be problematic in e.g. bridged rooms: You set some emotes from the matrix
side and a discord bridge would plop all the discord emotes in another pack in the same room.

MSC1951 defines a way to recommend using a pack of a different room - this MSC does not have an equivalent
of that. Instead, this MSC allows multiple emote packs for a room, and allows you to enable an emote
pack to be globally available for yourself accross all rooms you are in.

The core difference is how MSC1951 and this MSC define the emotes themselves: In MSC1951 you have to
set one state event *per emote*. While this might seem like a nice idea on the surface, it doesn't
scale well. There are people who easily use and want hundreds or even thousands of emotes accessible.
A simple dict of shortcode to mxc URI seems more appropriate for this.

Additionally, this MSC is already used in the wild since about two years by some clients, while MSC1951
only exists in MSC form, so far.

In general, MSC1951 feels like a heavier approach to emote sourcs, while this MSC is more lightweight
and thus should allow significantly larger packs.