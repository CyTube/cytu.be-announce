# Upcoming Adobe Flash Player End-of-Life

Adobe is [officially ending support for Flash Player on December 31,
2020](https://www.adobe.com/products/flashplayer/end-of-life.html).  In light of
this timeline, various web browser vendors are dropping support for Flash
browser plugins, which may impact certain user experiences on CyTube, and CyTube
is dropping support for certain legacy video types that depend on Flash
(accounting for ~0.05% of all video traffic on CyTube in the past 2 weeks).

If you make use of custom media types on CyTube (raw file support, custom media
manifests, rtmp, custom embeds), please read below.  **Officially supported
player types (such as YouTube) should not be impacted by this.**

## Dropping RTMP support

CyTube supports adding `rtmp://` URIs to embed [RTMP
streams](https://en.wikipedia.org/wiki/Real-Time_Messaging_Protocol), which are
played using a Flash-based player.  Due to the nature of this protocol, as far
as I am aware, there are no browser-native non-Flash players available.

CyTube will drop support for RTMP on December 31, 2020.  Users self-hosting
livestreams are recommended to migrate to HLS or DASH instead, both of which can
be natively played by modern browsers.

## Dropping `<object>`/`<embed>` tag support from custom embeds

CyTube supports adding custom embeds as a limited subset of HTML.  This feature
currently supports the `<iframe>` tag, which embeds an HTML iframe, and the
`<object>`/`<embed>` tags, which embed Flash objects.

CyTube will drop support for `<object>`/`<embed>` tags on December 31, 2020.
Users will continue to be able to embed `<iframe>` tags using this feature.

## Raw files / custom media manifests: unaffected

Users adding custom files as direct links or via custom media manifests with the
[supported MIME
types](https://github.com/calzoneman/sync/blob/3.0/docs/custom-media.md#acceptable-mime-types)
should not be impacted by the Flash EOL, since CyTube already uses an
HTML5-native player for these.
