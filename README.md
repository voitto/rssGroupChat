# RSS GroupChat Extension

**Version:** 1.1  
**Author:** Brian Hendrickson
**Date:** June 23, 2026

> **Changelog**
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

## Integration with MetaWebLog API

The RSS GroupChat Extension supports both creating and deleting replies in group conversations through XML-RPC requests to the feed URL.

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

## Security Considerations

- The `url` attribute should include appropriate authentication mechanisms, such as API keys or tokens, to ensure only authorized users can post to a group.
- Implementers should validate all content received via MetaWebLog API requests to prevent injection attacks.
- Applications should implement appropriate access controls to ensure private group conversations remain private.
- Delete operations should be restricted to the original author of the post to prevent unauthorized deletion of messages.
- Implementers should validate post IDs in delete requests to ensure they exist and belong to the authenticated user.
- Media uploaded via `metaWeblog.newMediaObject` should be authenticated identically to `newPost` (the key in the feed URL), and the host should enforce size and MIME-type limits before storing the bytes.
- Because group conversations are private, media `<enclosure>` URLs should be access-controlled rather than world-readable — for example time-limited signed URLs scoped to a single media object — so that possessing a feed item does not grant permanent, shareable access to the underlying file.
- A host should associate an uploaded media object with the message that references it (e.g. by confirming the `enclosure` `url` resolves to media the host itself stored for the authenticated user) rather than trusting an arbitrary external `url`.

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
    
    <item>
      <title>Welcome to our group chat!</title>
      <description>This is the first message in our group conversation.</description>
      <pubDate>Mon, 10 Mar 2025 15:00:00 GMT</pubDate>
      <link>https://example.com/messages/12344</link>
      <guid>https://example.com/messages/12344</guid>
      <author>user1@example.com</author>
      <groupchat:group id="group123" url="https://example.com/feed?key=userkey" title="Social Web Chat Group" />
    </item>
    
    <item>
      <title>RE: Welcome to our group chat!</title>
      <description>Thanks for setting this up!</description>
      <pubDate>Mon, 10 Mar 2025 15:30:00 GMT</pubDate>
      <link>https://example.com/messages/12345</link>
      <guid>https://example.com/messages/12345</guid>
      <author>user2@example.com</author>
      <groupchat:group id="group123" url="https://example.com/feed?key=userkey" title="Social Web Chat Group" />
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





