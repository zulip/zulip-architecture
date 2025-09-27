# Remote image thumbnailing

## Types of remote images

### Inline image URLs

These are `![Alt text](https://example.com/image.png)` images, which are
explicit[^1]. At message send time, we should render these as spinners in our
initial `rendered_content`[^2], and insert a background process to fetch them. If
the background fetch returns an image type, we should resize the image into our
set of supported sizes/types, and cache these in-memory[^3]. The duration of
this cache should be based on the caching headers of the remote image.

The message should then be silently updated to point to a signed `/thumbnail`
URL with a "reasonable" size/format; the signing covers the URL, but not the
size, since clients may rewrite it to their preferred size.

On the server side, the `/thumbnail` URL validates the signature, and returns
400 (possibly with the content of a static failure image, varying on `Accepts`
header) if the signature is invalid. It then checks that the requested
size/format is in its supported set, and "rounds" to the closest match if it is
not (based on `Accepts` header for format). If the thumbnail size/format is in
cache, it serves it.

If the requested image is not in our local cache, it must re-fetch it. This
happens synchronously, after which it must resize and store it in cache, and
then provide the appropriate resized image to the client. Any non-expired[^4]
size/format combinations should be re-rendered and inserted into the cache at
the same time, since the network fetch time is likely significant compared to
the resize time, we should endeavor to provide a consistent preview if the image
is mutating over time, and one access may herald other accesses from other
clients.

In the event that either the initial fetch, or subsequent re-fetches, times out,
returns a document with a non-image `Content-Type`, or cannot be parsed as its
purported image type, then we cache and return a stock "invalid image"
content. We may wish to set an upper time bound on this (or multiple different
bounds, based on the failure type), to handle intermittent failures.

The content requests must be made through Smokescreen, to ensure that they
cannot be redirected (via DNS or HTTP) into private IP space.

[^1]:
    We render these even if image previews are disabled, presumably? Since
    that's mostly about not fetching random network resources, not about
    preventing image uploads from rendering inline?

[^2]:
    How do we know how much space the spinner should take up? We do not know
    anything about the height of the returned image yet, and and yet need to
    choose a height that minimizes or avoides veritcal movement.

[^3]: In memcached? Or on disk, but then we need to do manual flushing of it?
[^4]: Possibly _all_ size/format combinations, for maximum consistency?

### Inline URLs

These are messages of the form:

```markdown
Look at my picture:

https://example.com/image.png
```

Or:

```markdown
[Look at my picture](https://example.com/image.png)
```

That is, a link (implied or explicit) with a URL which ends in an image
extension, assuming that image previews are enabled on the server and realm.

The extension provides a light implication that the URL is an image, which we
should inline. The above plan for inline image URLs holds, with the exception
that _nothing_ is inlined upon first message send, and in the event of failure
or non-image content, the message is not updated in any way[^5].

The effect of this is that intermittent failures of non-explicit image URLs is
that they are never retried if they initially fail.

[^5]:
    This means that these messages will grow taller after sending, which is a
    bad thing? We could also render them as spinners, and update to plain text
    if the request fails, which means users will be less likely to have vertical
    movement, but will see less information about the image until thumbnailing
    completes.

### Inline bare URLs

These are messages of the form:

```markdown
https://example.com/image.png
```

That is, a body entirely of a URL which ends in an image extension, assuming
that image previews are enabled on the server and realm.

These are treated as inline bare URLs, with the additional change that the
entire content of the message is silently updated with the thumbnailed image,
should it turn out to actually be an image.

### Opengraph images

These are messages of the form:

```markdown
https://example.com/
```

...where `example.com` has `og:...` tags which we can preview, assuming that
`INLINE_URL_EMBED_PREVIEW` is enabled and the realm has URL previews enabled.

Any images from this preview will be treated as "Inline image URLs", above.

## Effects on existing URL endpoints

### `/thumbnail?url=...&size=...`

Existing `/thumbnail` URLs are of the form:

    /thumbnail?url=user_uploads%2F2%2F85%2FXoqF0K7XEOLVGylgdpof80RB%2Fimg.png&size=full
    /thumbnail?url=user_uploads%2F2%2F85%2FXoqF0K7XEOLVGylgdpof80RB%2Fimg.png&size=thumbnail

    /thumbnail?url=https%3A%2F%2Fwww.example.com%2Fimages%2Ffilename.png&size=full
    /thumbnail?url=https%3A%2F%2Fwww.example.com%2Fimages%2Ffilename.png&size=thumbnail

These were only generated by `THUMBNAIL_IMAGES = True` servers; they may appear
in historical messages even if it is not currently set.

The former two are currently serve the full-size `/user_uploads/` equivalents,
regardless of `size`. The latter two accepted unauthenticated unsigned requests,
and were not rate-limited; for security reasons, they currently always 401.

The endpoint will begin supporting _signed_ URL requests. These sign the `url`
parameter, and allow the caller to adjust the size and format of the response.
See "Inline image URLs", above.

### `/external_content/...`

Since thumbnailing is performed on all remote images, there is no need for Camo
for images in new messages anymore; all images are served either through
`/user_uploads/...` or `/thumbnail?...`

However, until videos are rendered as their server-side-generated thumbnails,
videos must continue to go through Camo; previous messages also still encode
`/external_content/` URLs, which should still be served.

So for backwards-compatibility, the Camo server should be preserved for now, and
continue to serve `/external_content/` URLs.
