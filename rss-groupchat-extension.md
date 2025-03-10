# RSS GroupChat Extension

**Version:** 1.0  
**Author:** Brian Hendrickson
**Date:** March 10, 2025

## Overview

The RSS GroupChat Extension is a specification that extends RSS 2.0 to support group chat functionality within feed readers. This extension enables feed readers to segregate private conversation posts from public news posts and allows participants to reply to specific conversations using the MetaWebLog API.

## Namespace Declaration

The GroupChat Extension uses the following namespace:

```
xmlns:groupchat="https://rss.ag/rss-groupchat-extension/ns/1.0/"
```

This namespace must be declared in the `<rss>` element of any feed implementing this extension:

```xml
<rss version="2.0" xmlns:groupchat="https://github.com/rss.ag/rss-groupchat-extension/ns/1.0/">
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

## Integration with MetaWebLog API

### Sending Replies to Group Conversations

Clients can send replies to group conversations by making a MetaWebLog API request to the URL specified in the `url` attribute of the `<groupchat:group>` element. The group's `id` value must be included in the `categories` field of the MetaWebLog API request.

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

## Security Considerations

- The `url` attribute should include appropriate authentication mechanisms, such as API keys or tokens, to ensure only authorized users can post to a group.
- Implementers should validate all content received via MetaWebLog API requests to prevent injection attacks.
- Applications should implement appropriate access controls to ensure private group conversations remain private.

## Compatibility

This extension is designed to be fully compatible with RSS 2.0 and can be implemented alongside other RSS extensions. Feed readers that do not support this extension will simply ignore the additional elements and display the items as regular feed entries.

## Example Feed

Below is a complete example of an RSS feed implementing the GroupChat Extension:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rss version="2.0" xmlns:groupchat="https://github.com/rss.ag/rss-groupchat-extension/ns/1.0/">
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

1. RSS 2.0 Specification: https://www.rssboard.org/rss-specification
2. MetaWebLog API Specification: http://xmlrpc.scripting.com/metaWeblogApi.html





