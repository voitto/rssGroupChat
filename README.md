# RSS GroupChat Extension

**Version:** 1.4  
**Author:** Brian Hendrickson
**Date:** August 1, 2026

> **Changelog**
> - **1.4** — Added emoji reactions (tapbacks): an item-level
>   `<groupchat:reaction>` element carries the message's current set of
>   emoji reactions (one element per reacting member, keyed by the same
>   stable author ids rosters use), and a new `groupchat.setReaction`
>   XML-RPC method lets a remote participant set, replace, or clear their
>   reaction on a message they can read. Reactions follow the same
>   snapshot contract as icons and rosters: the elements present on an
>   item at fetch time are that item's complete current reaction set, and
>   absence carries removal. Backward compatible: the namespace URI is
>   unchanged, and readers that do not implement reactions simply ignore
>   the elements.
> - **1.3** — Added member rosters and stable author ids: a channel-level
>   `<groupchat:roster>` element carries each group's member snapshot
>   (names, per-member avatar URLs, per-viewer self flag), and an
>   item-level `<groupchat:authorId>` gives every message a stable
>   per-person key to hang member identity on. Also documents the
>   per-viewer `<groupchat:isItemOwner>` element that implementations have
>   carried since 1.0. Backward compatible: the namespace URI is
>   unchanged, and readers that do not implement rosters simply ignore the
>   elements.
> - **1.2** — Added group icons: a host can convey a group conversation's
>   icon (an emoji, or an access-controlled photo URL) on each item via the
>   new `<groupchat:icon>` element, so subscribing readers can render the
>   same icon the host's own users see. Backward compatible: the namespace
>   URI is unchanged, and readers that do not implement icons simply ignore
>   the element.
> - **1.1** — Added media attachments: replies may carry an image or video,
>   uploaded with the standard MetaWebLog `metaWeblog.newMediaObject` call and
>   referenced from `metaWeblog.newPost` via a standard RSS `<enclosure>`.
>   Backward compatible: the namespace URI is unchanged, and readers that do
>   not implement media simply ignore the `enclosure`.
> - **1.0** — Initial release.

## Overview

The RSS GroupChat Extension is a specification that extends RSS 2.0 to support group chat functionality within feed readers. This extension enables feed readers to segregate private conversation posts from public news posts and allows participants to reply to specific conversations using the MetaWebLog API. Replies may include media (images and video), transferred with the standard MetaWebLog media-upload call and surfaced in the feed as a standard RSS `<enclosure>`.

## Namespace Declaration

The GroupChat Extension uses the following namespace:

```
xmlns:groupchat="https://rss.ag/rss-groupchat-extension/ns/1.0/"
```

This namespace must be declared in the `<rss>` element of any feed implementing this extension:

```xml
<rss version="2.0" xmlns:groupchat="https://rss.ag/rss-groupchat-extension/ns/1.0/">
```

## Elements and Attributes

### `<groupchat:group>` Element

The `<groupchat:group>` element must be a child of the `<item>` element in an RSS feed. This element identifies an item as part of a group conversation.

#### Required Attributes:

- **id** (required): A string that uniquely identifies the particular group chat on the host feed reader/publisher web app.
- **url** (required): The URL of the feed that hosts this group conversation. This URL serves as the endpoint for MetaWebLog API requests.
- **title** (required): A human-readable name for the group conversation.

#### Example:

```xml
<item>
  <title>New message in group chat</title>
  <description>This is the content of the message in the group chat.</description>
  <pubDate>Mon, 10 Mar 2025 15:30:00 GMT</pubDate>
  <link>https://example.com/messages/12345</link>
  <guid>https://example.com/messages/12345</guid>
  <groupchat:group id="group123" url="https://example.com/feed?key=userkey" title="Social Web Chat Group" />
</item>
```

### `<enclosure>` Element (Media)

An item whose message carries an image or video includes a standard RSS 2.0 `<enclosure>` element alongside its `<groupchat:group>`. No new namespace is involved — media reuses the RSS 2.0 enclosure mechanism that podcast feeds already use.

#### Attributes:

- **url** (required): A fetchable URL for the media on the host. Because group conversations are private, this URL should be access-controlled — for example a time-limited signed URL that does not require the fetching client to hold a session on the host.
- **type** (required): The MIME type of the media (e.g. `image/jpeg`, `video/mp4`).
- **length** (recommended): The size of the media in bytes, per RSS 2.0.

#### Example:

```xml
<item>
  <title>(media)</title>
  <description></description>
  <enclosure url="https://example.com/api/media/8c0dbeb6-f2ad-4057-bbff-c430f930bfc5?exp=1789200000&amp;sig=Yt3v...Qk" type="image/jpeg" length="1816742" />
  <pubDate>Mon, 10 Mar 2025 15:30:00 GMT</pubDate>
  <guid isPermaLink="false">15</guid>
  <author>user2@example.com</author>
  <groupchat:group id="group123" url="https://example.com/feed?key=userkey" title="Social Web Chat Group" />
</item>
```

A media-only message (no accompanying text) uses the placeholder `(media)` as a human-readable `<title>` so plain RSS readers show something sensible. GroupChat-aware clients should render the enclosure and ignore that placeholder title when an `<enclosure>` is present.

### `<groupchat:icon>` Element (Group Icon)

A group conversation may have an icon — a photo or an emoji — chosen by the group's owner on the host. The `<groupchat:icon>` element conveys that icon to subscribing readers so every participant's client can render the same visual identity for the conversation, regardless of which server they connect through.

The element is an optional child of `<item>`, appearing alongside the item's `<groupchat:group>` element. It describes the icon of the *group* the item belongs to, not of the item itself.

#### Attributes (exactly one of `emoji` or `url`):

- **emoji**: A short emoji string that is the group's icon (e.g. `emoji="🎉"`).
- **url**: A fetchable URL for the group's icon image on the host. Because group conversations are private, this URL should be access-controlled — for example a time-limited signed URL, exactly like a media `<enclosure>` — rather than world-readable.
- **type** (optional, only with `url`): The MIME type of the icon image (e.g. `image/jpeg`).

#### Example:

```xml
<item>
  <title>New message in group chat</title>
  <description>This is the content of the message in the group chat.</description>
  <pubDate>Mon, 10 Mar 2025 15:30:00 GMT</pubDate>
  <guid>https://example.com/messages/12345</guid>
  <groupchat:group id="group123" url="https://example.com/feed?key=userkey" title="Social Web Chat Group" />
  <groupchat:icon emoji="🎉" />
</item>
```

or, for a photo icon:

```xml
<groupchat:icon url="https://example.com/api/media/9d1acfd2-01bb-4c11-a2e0-5f2b1c930bfd?exp=1789200000&amp;sig=Xy7w...Rm" type="image/jpeg" />
```

#### Snapshot semantics

The icon element is a snapshot of the group's *current* icon at the time the feed is rendered:

- A host that supports icons MUST include the element on **every** item belonging to a group that currently has an icon, and MUST omit it on every item belonging to a group that does not. All items of one group within a single feed document therefore agree.
- A subscriber may adopt the icon from any item of the group; the value seen on the most recent fetch is the group's current icon. If the latest fetch carries **no** `<groupchat:icon>` on a group's items, the group currently has no custom icon and the reader should fall back to its default rendering (initials, glyph, etc.).
- When the owner changes or removes the icon, the host SHOULD notify its rssCloud subscribers (the same mechanism used for new messages) so readers re-fetch and pick up the new icon promptly, even when no new message accompanies the change.
- Because a signed `url` expires, hosts SHOULD mint a fresh signed URL each time the feed is rendered, and subscribers SHOULD refresh their stored copy of the URL on every fetch. A reader that has not fetched the feed within the URL's validity window should degrade gracefully (e.g. fall back to its default rendering) until the next fetch.

#### End-to-end encrypted groups

The icon — like the group's `title` — is conversation *metadata*, not message content. In deployments where message bodies are end-to-end encrypted, the icon still travels in the clear so relaying servers can render conversation lists. Hosts for whom that is unacceptable MAY simply omit the element for encrypted groups.

### `<groupchat:roster>` Element (Member Roster)

A feed that carries group conversations SHOULD also carry, at the **channel** level, one `<groupchat:roster>` element per group the feed's reader belongs to. The roster is the group's member snapshot at feed-render time: who is in the conversation, what to call them, and (optionally) a fetchable photo for each. It is what lets a subscribing reader show member lists and profile photos for people whose accounts live on the host — without the roster, a subscriber only ever sees free-text `<author>` strings.

#### Attributes:

- **id** (required): The group's identifier — the same value items carry in their `<groupchat:group>` element, which is how a reader routes the roster to the right conversation.
- **title** (optional): The group's human-readable name as rendered for this feed's reader. Feeds are per-reader, so a host that derives names from member lists SHOULD exclude the reader themselves, exactly as it does for the `<groupchat:group>` title. Carrying it here means a conversation with no messages yet still subscribes under a real name.

#### `<groupchat:member>` children (one per member):

- **id** (required): The host's stable identifier for this member (an integer user id on the host). This is the key that item-level `<groupchat:authorId>` values match, and it is stable across display-name changes.
- **name** (required): The member's display name.
- **avatar** (optional): A fetchable URL for the member's profile photo on the host. Because group conversations are private, this URL should be access-controlled — a time-limited signed URL, exactly like a media `<enclosure>` or a photo `<groupchat:icon>` — and hosts SHOULD mint a fresh signed URL on each render. A member without a photo simply omits the attribute.
- **self** (optional): `"true"` on the single member entry that is the feed's reader themselves. Computed per viewer like `<groupchat:isItemOwner>`; readers use it to know which entry is "you" without name matching. Omitted otherwise.
- **ai** (optional): `"true"` when the member is a host-operated automated participant (an AI assistant), so readers can badge it. Omitted otherwise.

#### Example:

```xml
<channel>
  <title>Social Web messages — Erik</title>
  <cloud domain="example.com" port="443" path="/rsscloud/pleaseNotify" registerProcedure="" protocol="http-post" />
  <groupchat:roster id="group123" title="Brian &amp; Evan">
    <groupchat:member id="1" name="Brian" avatar="https://example.com/api/media/660f6467-b68b-475a-81f8-29116da0dbcb?exp=1789200000&amp;sig=r8Xk...OU" />
    <groupchat:member id="12" name="Evan Kendig" avatar="https://example.com/api/media/9a1c2f4e-8d3b-4c5a-9e6f-7a8b9c0d1e2f?exp=1789200000&amp;sig=Gf7c...Yw" />
    <groupchat:member id="17" name="Erik Solano" self="true" />
  </groupchat:roster>
  <!-- items follow -->
</channel>
```

#### Snapshot semantics

The roster follows the same snapshot contract as `<groupchat:icon>`:

- The roster is the group's **complete** current membership at render time. A member absent from the latest snapshot has left the group; an `avatar` attribute absent from a member's entry means that member currently has no photo. Absence carries removal in both cases — there is no separate tombstone.
- A host SHOULD emit a roster for **every** group the feed's reader belongs to, including groups with no items in the feed window, so brand-new conversations carry member identity from their first fetch.
- When a member's profile photo changes (or is removed), the host SHOULD notify its rssCloud subscribers — the same mechanism used for new messages and icon changes — so readers re-fetch promptly even when no message accompanies the change. Note the fan-out: a member's photo appears in the rosters of every feed whose reader shares a group with them, so the host notifies the subscribers of each such feed.
- Signed `avatar` URLs expire like signed icon URLs: subscribers SHOULD refresh their stored copies on every fetch and degrade gracefully (initials, glyph) when a URL expires between fetches.

#### End-to-end encrypted groups

Membership — like the group's title and icon — is conversation *metadata*, not message content. Rosters are carried for encrypted groups too; deployments that derive group titles from member names already expose this information. Hosts for whom that is unacceptable MAY omit rosters for encrypted groups.

### `<groupchat:authorId>` Element (Stable Author Key)

An optional child of `<item>`: the host's stable identifier for the message's author, matching a `<groupchat:member>` `id` in the item's group's roster. The RSS `<author>` element remains a free-text display string for plain-reader compatibility; `<groupchat:authorId>` is what GroupChat-aware readers key member identity (names, avatars) on.

```xml
<item>
  <title>doing pretty well!</title>
  <description>doing pretty well!</description>
  <pubDate>Mon, 10 Mar 2025 15:30:00 GMT</pubDate>
  <guid isPermaLink="false">36</guid>
  <author>Evan Kendig</author>
  <groupchat:group id="group123" url="https://example.com/feed?key=userkey" title="Social Web Chat Group" />
  <groupchat:isItemOwner>false</groupchat:isItemOwner>
  <groupchat:authorId>12</groupchat:authorId>
</item>
```

Encrypted-group items (whose `<author>` is a placeholder) SHOULD carry `<groupchat:authorId>` too: sender identity is routing metadata in the encrypted model, and it lets readers resolve roster avatars without waiting for decryption.

### `<groupchat:reaction>` Element (Emoji Reactions)

A message may carry emoji reactions — the small "tapback" acknowledgements chat users attach to a specific message (a thumbs-up, a heart) without composing a reply. The `<groupchat:reaction>` element conveys them: an optional, repeatable child of `<item>`, one element per member who currently has a reaction on that message.

#### Attributes:

- **emoji** (required): The reaction itself. SHOULD be a single emoji grapheme (which may span several code points — skin tones, `‼️`, ZWJ sequences). Hosts SHOULD reject long or control-character values; readers SHOULD render the value as an opaque short string.
- **authorId** (required): The host's stable identifier for the reacting member — the same key space as `<groupchat:authorId>` and roster `<groupchat:member>` `id` values, so readers resolve the reactor's name and avatar from the roster with no extra machinery.
- **author** (optional): The reacting member's display name at render time, for readers that have no roster in hand.
- **self** (optional): `"true"` when the reacting member is the feed's reader themselves. Computed per viewer exactly like `<groupchat:isItemOwner>` and the roster's `self` attribute; readers use it to highlight "your" reaction and to know what a repeated `setReaction` will replace. Omitted otherwise.

#### Example:

```xml
<item>
  <title>OK. I'll come by.</title>
  <description>OK. I'll come by.</description>
  <pubDate>Mon, 10 Mar 2025 15:30:00 GMT</pubDate>
  <guid isPermaLink="false">36</guid>
  <author>Evan Kendig</author>
  <groupchat:group id="group123" url="https://example.com/feed?key=userkey" title="Social Web Chat Group" />
  <groupchat:isItemOwner>false</groupchat:isItemOwner>
  <groupchat:authorId>12</groupchat:authorId>
  <groupchat:reaction emoji="👍" authorId="1" author="Brian" self="true" />
  <groupchat:reaction emoji="❤️" authorId="17" author="Erik Solano" />
</item>
```

#### Snapshot semantics

Reactions follow the same snapshot contract as `<groupchat:icon>` and `<groupchat:roster>`, applied per item:

- The `<groupchat:reaction>` elements present on an item are that message's **complete** current reaction set at render time. A member appears **at most once** per item; setting a new reaction replaces the old one, and a member absent from the latest snapshot has removed their reaction. Absence carries removal — there is no separate tombstone. A reader that encounters duplicate `authorId` values on one item SHOULD keep only the last.
- Because items are conventionally treated as immutable by RSS readers, GroupChat-aware readers SHOULD re-reconcile the reaction set of **already-seen items** on every fetch (the same way they re-check `<groupchat:icon>`), rather than skipping known guids wholesale.
- When a reaction is set, replaced, or removed, the host SHOULD notify its rssCloud subscribers — the same mechanism used for new messages, icon changes, and roster changes — so readers re-fetch promptly even though no new item accompanies the change. Note the fan-out: the item appears in the feed of every member of the group, so the host notifies the subscribers of each member's feed.
- Reactions travel only while their item is present in the feed document. A reaction to a message that has already scrolled out of the host's feed window does not propagate; hosts SHOULD size their window so that the recent messages users actually react to are still present.

#### End-to-end encrypted groups

Encrypted items MAY carry `<groupchat:reaction>` elements too: the reaction (emoji, reactor id, per-viewer `self`) travels in the clear as *acknowledgement metadata* — alongside the sender id, timing, and group routing that the encrypted model already exposes — while the message content it points at stays opaque. This is the pragmatic reading of the icon/roster metadata rule, and it is what lets a reaction survive relaying servers that cannot decrypt anything.

Deployments for whom the emoji itself is sensitive MAY simply omit the element for encrypted groups (the same escape hatch icons and rosters have); a fully sealed alternative — the reaction as an encrypted application message inside the group's opaque transport (e.g. a `<groupchat:mls>` payload) — remains implementation-defined in this revision.

### `<groupchat:isItemOwner>` Element

Carried by implementations since 1.0 and documented here for completeness: an optional child of `<item>` whose value is `true` when the item was authored by the feed's reader themselves, `false` otherwise. Feeds are per-reader, so the value is computed relative to the feed's key. Readers use it to render the reader's own messages on the sending side of a chat UI without re-deriving identity — and, since 1.3, the roster's `self` attribute follows the same per-viewer rule.

### Implementation notes

- **Parse by local name.** The reference implementation (Social Web v3) matches extension elements by their local name (`roster`, `member`, `authorId`, `icon`, ...) rather than by resolved namespace URI, and emits the namespace declaration `xmlns:groupchat="https://socialweb.cloud/rss/groupchat"`. Interoperating parsers SHOULD match local names so feeds using either namespace URI string parse identically.
- **`<groupchat:group>` forms.** The attribute form shown in this document (`<groupchat:group id=".." url=".." title=".." />`) and a child-element form (`<groupchat:group><id>..</id><title>..</title></groupchat:group>`, emitted by Social Web v3) both exist in the wild. Parsers SHOULD accept both.
- **Encrypted transport elements.** Social Web v3 additionally carries `<groupchat:mls>` (opaque end-to-end-encrypted payloads with `id`/`epoch`/`contentType`/`ciphertext` children), `<groupchat:connect>` (a pending join link), and `<groupchat:deleted>` (a group-deletion tombstone item). These are implementation-defined in 1.3 and may be standardized in a future revision; readers that do not implement them ignore them safely.

## Integration with MetaWebLog API

The RSS GroupChat Extension supports creating and deleting replies in group conversations — and, since 1.4, setting and clearing emoji reactions on them — through XML-RPC requests to the feed URL.

### Sending Replies to Group Conversations

Clients can send replies to group conversations by making a MetaWebLog API request to the URL specified in the `url` attribute of the `<groupchat:group>` element. The group's `id` value must be included in the `categories` field of the MetaWebLog API request.
`blogid`, `username` and `password` are ignored for now (key in URL provides auth) but are included for well-formed XMLRPC.
The final boolean parameter indicates whether the post should be published (typically `true` for delete operations).

#### MetaWebLog Request Example:

```xml
<?xml version="1.0"?>
<methodCall>
  <methodName>metaWeblog.newPost</methodName>
  <params>
    <param>
      <value><string>blogid</string></value>
    </param>
    <param>
      <value><string>username</string></value>
    </param>
    <param>
      <value><string>password</string></value>
    </param>
    <param>
      <value>
        <struct>
          <member>
            <name>title</name>
            <value><string>Reply to group chat</string></value>
          </member>
          <member>
            <name>description</name>
            <value><string>This is my reply to the group conversation.</string></value>
          </member>
          <member>
            <name>categories</name>
            <value>
              <array>
                <data>
                  <value><string>group123</string></value>
                </data>
              </array>
            </value>
          </member>
        </struct>
      </value>
    </param>
    <param>
      <value><boolean>1</boolean></value>
    </param>
  </params>
</methodCall>
```

### Attaching Media to Replies

A reply may carry an image or video. Media transfer uses the standard MetaWebLog `metaWeblog.newMediaObject` call, performed *alongside* the `metaWeblog.newPost` that creates the message — the same two-call pattern blogging clients use to attach an image to a post. Both calls go to the conversation's feed `url` and are authenticated the same way (the key in the URL).

The sequence is:

1. **Upload the binary** with `metaWeblog.newMediaObject`. The struct parameter carries the file `name`, MIME `type`, and the raw file `bits` (base64-encoded). The host stores the media and returns a struct containing a `url` at which the media can be fetched.
2. **Create the message** with `metaWeblog.newPost`, adding a standard RSS `enclosure` member to the struct whose `url` is the value returned in step 1. The host attaches that media to the new message. A media-only message is valid — send an empty `title`/`description` with the `enclosure` member present.

The host then republishes the message in its feed with a matching `<enclosure>` element (see [`<enclosure>` Element (Media)](#enclosure-element-media)), so every subscriber — including the original sender, who sees the message return on the round-trip — can render the attachment.

Separating the upload (`newMediaObject`) from the message (`newPost`) keeps message transmission unchanged for text-only replies and lets the host enforce media size/type limits and persist the blob before the message that references it exists.

#### `metaWeblog.newMediaObject` Request Example:

```xml
<?xml version="1.0"?>
<methodCall>
  <methodName>metaWeblog.newMediaObject</methodName>
  <params>
    <param>
      <value><string>blogid</string></value>
    </param>
    <param>
      <value><string>username</string></value>
    </param>
    <param>
      <value><string>password</string></value>
    </param>
    <param>
      <value>
        <struct>
          <member>
            <name>name</name>
            <value><string>photo.jpg</string></value>
          </member>
          <member>
            <name>type</name>
            <value><string>image/jpeg</string></value>
          </member>
          <member>
            <name>bits</name>
            <value><base64>/9j/4AAQSkZJRgABAQ... (base64-encoded file bytes) ...</base64></value>
          </member>
        </struct>
      </value>
    </param>
  </params>
</methodCall>
```

#### `metaWeblog.newMediaObject` Response:

The response is a struct with a single `url` member, per the MetaWebLog specification. The returned URL is what the subsequent `newPost` references in its `enclosure`.

```xml
<?xml version="1.0"?>
<methodResponse>
  <params>
    <param>
      <value>
        <struct>
          <member>
            <name>url</name>
            <value><string>https://example.com/api/media/8c0dbeb6-f2ad-4057-bbff-c430f930bfc5?exp=1789200000&amp;sig=Yt3v...Qk</string></value>
          </member>
        </struct>
      </value>
    </param>
  </params>
</methodResponse>
```

#### `metaWeblog.newPost` with an `enclosure` member:

The message-creating call is identical to a text reply except for an added `enclosure` member in the struct. Its value is a struct carrying the `url` returned by `newMediaObject` (and, for well-formed RSS, the `type` and `length`):

```xml
        <struct>
          <member>
            <name>title</name>
            <value><string>Check out this photo!</string></value>
          </member>
          <member>
            <name>categories</name>
            <value>
              <array>
                <data>
                  <value><string>group123</string></value>
                </data>
              </array>
            </value>
          </member>
          <member>
            <name>enclosure</name>
            <value>
              <struct>
                <member>
                  <name>url</name>
                  <value><string>https://example.com/api/media/8c0dbeb6-f2ad-4057-bbff-c430f930bfc5?exp=1789200000&amp;sig=Yt3v...Qk</string></value>
                </member>
                <member>
                  <name>type</name>
                  <value><string>image/jpeg</string></value>
                </member>
                <member>
                  <name>length</name>
                  <value><int>1816742</int></value>
                </member>
              </struct>
            </value>
          </member>
        </struct>
```

### Deleting Replies from Group Conversations

Clients can delete their own replies from group conversations by making an XML-RPC request to the URL specified in the `url` attribute of the `<groupchat:group>` element. The delete operation requires only the post ID of the message to be deleted.

#### XML-RPC Delete Request Example:

```xml
<?xml version="1.0"?>
<methodCall>
  <methodName>metaWeblog.deletePost</methodName>
  <params>
    <param>
      <value><string>postid</string></value>
    </param>
    <param>
      <value><string>username</string></value>
    </param>
    <param>
      <value><string>password</string></value>
    </param>
    <param>
      <value><boolean>1</boolean></value>
    </param>
  </params>
</methodCall>
```

Where:
- `postid` is the unique identifier of the post to be deleted
- `blogid`, `username` and `password` are ignored for now (key in URL provides auth) but are included for well-formed XMLRPC
- The final boolean parameter indicates whether the post should be published (typically `true` for delete operations)

### Reacting to Messages in Group Conversations

Since 1.4, a participant can set, replace, or clear their emoji reaction on a message by making an XML-RPC request to the URL specified in the `url` attribute of the `<groupchat:group>` element, using the `groupchat.setReaction` method. The call is deliberately shaped like `metaWeblog.deletePost` — a post id plus the well-formed-XMLRPC stub credentials — with the emoji as the final parameter.

The operation is a **replacement**: a caller has at most one reaction per message, a new emoji replaces any previous one, and an **empty string clears** the caller's reaction. Repeating a call is idempotent.

#### `groupchat.setReaction` Request Example:

```xml
<?xml version="1.0"?>
<methodCall>
  <methodName>groupchat.setReaction</methodName>
  <params>
    <param>
      <value><string>postid</string></value>
    </param>
    <param>
      <value><string>username</string></value>
    </param>
    <param>
      <value><string>password</string></value>
    </param>
    <param>
      <value><string>👍</string></value>
    </param>
  </params>
</methodCall>
```

Where:
- `postid` is the unique identifier of the message being reacted to — the item's `<guid>` value in the group's feed. For an end-to-end encrypted item this is the namespaced guid as emitted (e.g. `mls-3-42`); hosts resolve either form to the right message
- `blogid`, `username` and `password` are ignored for now (key in URL provides auth) but are included for well-formed XMLRPC
- The final string parameter is the emoji to set; an empty string clears the caller's existing reaction

#### Response:

The response is a single boolean, like a delete:

```xml
<?xml version="1.0"?>
<methodResponse>
  <params>
    <param>
      <value><boolean>1</boolean></value>
    </param>
  </params>
</methodResponse>
```

After accepting a reaction change the host re-renders its feeds with the updated `<groupchat:reaction>` snapshot on the affected item and notifies its rssCloud subscribers, exactly as it does for a new message. The reacting participant sees their own reaction return on the round-trip (`self="true"` in their feed), which confirms the write without a separate read API.

## Real-time Notifications

### Using Cloud Element

For real-time updates, this extension recommends using the standard RSS `<cloud>` element to notify clients of feed updates:

```xml
<cloud domain="rpc.example.com" port="80" path="/RPC2" registerProcedure="pleaseNotify" protocol="xml-rpc" />
```

When the feed is updated with new group chat messages, the cloud mechanism sends a notification to subscribed clients, prompting them to fetch the updated feed.

### WebSocket Implementation

While not part of the core RSS specification, this extension recommends implementing WebSockets to push updates from the client feed reader to the browser for immediate rendering. This is handled at the application level and is not part of the feed format itself.

## Validation

A valid RSS feed implementing the GroupChat Extension must:

1. Declare the GroupChat namespace in the `<rss>` element
2. Include `<groupchat:group>` elements only within `<item>` elements
3. Ensure all required attributes (`id`, `url`, and `title`) are present on each `<groupchat:group>` element
4. For items carrying media, include a standard RSS `<enclosure>` with at least `url` and `type`; the enclosure `url` must be fetchable by subscribers (subject to the access controls below)
5. Include `<groupchat:icon>` elements only within `<item>` elements that also carry a `<groupchat:group>`, with exactly one of the `emoji` or `url` attributes present; when a group has an icon, carry the element consistently on every item of that group in the document
6. Include `<groupchat:reaction>` elements only within `<item>` elements that also carry a `<groupchat:group>`, each with `emoji` and `authorId` present and at most one element per `authorId` per item

## Security Considerations

- The `url` attribute should include appropriate authentication mechanisms, such as API keys or tokens, to ensure only authorized users can post to a group.
- Implementers should validate all content received via MetaWebLog API requests to prevent injection attacks.
- Applications should implement appropriate access controls to ensure private group conversations remain private.
- Delete operations should be restricted to the original author of the post to prevent unauthorized deletion of messages.
- Implementers should validate post IDs in delete requests to ensure they exist and belong to the authenticated user.
- Media uploaded via `metaWeblog.newMediaObject` should be authenticated identically to `newPost` (the key in the feed URL), and the host should enforce size and MIME-type limits before storing the bytes.
- Because group conversations are private, media `<enclosure>` URLs should be access-controlled rather than world-readable — for example time-limited signed URLs scoped to a single media object — so that possessing a feed item does not grant permanent, shareable access to the underlying file.
- A host should associate an uploaded media object with the message that references it (e.g. by confirming the `enclosure` `url` resolves to media the host itself stored for the authenticated user) rather than trusting an arbitrary external `url`.
- A `<groupchat:icon>` `url` should be access-controlled the same way an `<enclosure>` `url` is (e.g. a time-limited signed URL), and should only ever reference media the host itself stores for the group; readers should treat the icon `url` as untrusted remote content (fetch it as an image only, never follow it as a navigation target).
- Because the icon (like the group `title`) is conveyed as plaintext metadata even for end-to-end encrypted conversations, hosts and clients should not place secret content in group icons.
- `groupchat.setReaction` should be restricted to members of the group that owns the referenced message, authenticated identically to `newPost` (the key in the feed URL), and hosts should validate that the post id exists and is visible to the caller before writing.
- Hosts should validate the reaction value (length-limit it, reject control characters) before storing or re-emitting it, and readers should render reaction strings as opaque inert text — never as markup.

## Compatibility

This extension is designed to be fully compatible with RSS 2.0 and can be implemented alongside other RSS extensions. Feed readers that do not support this extension will simply ignore the additional elements and display the items as regular feed entries.

## Example Feed

Below is a complete example of an RSS feed implementing the GroupChat Extension:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rss version="2.0" xmlns:groupchat="https://rss.ag/rss-groupchat-extension/ns/1.0/">
  <channel>
    <title>Example Feed with GroupChat Extension</title>
    <link>https://example.com/feed</link>
    <description>This feed demonstrates the GroupChat Extension for RSS 2.0</description>
    <language>en-us</language>
    <pubDate>Mon, 10 Mar 2025 16:00:00 GMT</pubDate>
    <lastBuildDate>Mon, 10 Mar 2025 16:00:00 GMT</lastBuildDate>
    <cloud domain="rpc.example.com" port="80" path="/RPC2" registerProcedure="pleaseNotify" protocol="xml-rpc" />
    <groupchat:roster id="group123" title="Social Web Chat Group">
      <groupchat:member id="1" name="User One" avatar="https://example.com/api/media/660f6467-b68b-475a-81f8-29116da0dbcb?exp=1789200000&amp;sig=r8Xk...OU" />
      <groupchat:member id="2" name="User Two" self="true" />
    </groupchat:roster>
    
    <item>
      <title>Welcome to our group chat!</title>
      <description>This is the first message in our group conversation.</description>
      <pubDate>Mon, 10 Mar 2025 15:00:00 GMT</pubDate>
      <link>https://example.com/messages/12344</link>
      <guid>https://example.com/messages/12344</guid>
      <author>user1@example.com</author>
      <groupchat:group id="group123" url="https://example.com/feed?key=userkey" title="Social Web Chat Group" />
      <groupchat:icon emoji="🎉" />
      <groupchat:isItemOwner>false</groupchat:isItemOwner>
      <groupchat:authorId>1</groupchat:authorId>
    </item>
    
    <item>
      <title>RE: Welcome to our group chat!</title>
      <description>Thanks for setting this up!</description>
      <pubDate>Mon, 10 Mar 2025 15:30:00 GMT</pubDate>
      <link>https://example.com/messages/12345</link>
      <guid>https://example.com/messages/12345</guid>
      <author>user2@example.com</author>
      <groupchat:group id="group123" url="https://example.com/feed?key=userkey" title="Social Web Chat Group" />
      <groupchat:icon emoji="🎉" />
      <groupchat:isItemOwner>true</groupchat:isItemOwner>
      <groupchat:authorId>2</groupchat:authorId>
      <groupchat:reaction emoji="👍" authorId="1" author="User One" />
    </item>
    
    <item>
      <title>(media)</title>
      <description></description>
      <enclosure url="https://example.com/api/media/8c0dbeb6-f2ad-4057-bbff-c430f930bfc5?exp=1789200000&amp;sig=Yt3v...Qk" type="image/jpeg" length="1816742" />
      <pubDate>Mon, 10 Mar 2025 15:45:00 GMT</pubDate>
      <link>https://example.com/messages/12346</link>
      <guid isPermaLink="false">12346</guid>
      <author>user2@example.com</author>
      <groupchat:group id="group123" url="https://example.com/feed?key=userkey" title="Social Web Chat Group" />
      <groupchat:icon emoji="🎉" />
    </item>
    
    <item>
      <title>Public news post</title>
      <description>This is a regular news post, not part of any group chat.</description>
      <pubDate>Mon, 10 Mar 2025 14:00:00 GMT</pubDate>
      <link>https://example.com/news/98765</link>
      <guid>https://example.com/news/98765</guid>
      <author>admin@example.com</author>
    </item>
  </channel>
</rss>
```

## License

MIT

## References

1. RSS 2.0 Specification: https://cyber.harvard.edu/rss/rss.html
2. MetaWebLog API Specification: http://xmlrpc.scripting.com/metaWeblogApi.html


## Implementations

The following implementations of the RSS GroupChat Extension are available:

1. [RSS GroupChat Server](https://github.com/voitto/rss-groupchat-server) - Example server implementation
2. [RSS GroupChat Client](https://github.com/voitto/rss-groupchat-client) - Example client implementation
3. [Social Web](https://socialweb.cloud/) - Production implementation in a feed reader with group chat support 





