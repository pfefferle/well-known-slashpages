# `.well-known/slashpages` resource identifier

This document describes the use of URIs whose path component is `/.well-known/slashpages`.

## Registration

Information required by RFC 8615:

```
URI suffix: slashpages
Change controller: Matthias Pfefferle <https://notiz.blog/>
Specification document(s): https://github.com/pfefferle/well-known-slashpages
Status: provisional
Related information: Slash Pages <https://slashpages.net/>, OPML 2.0 specification <http://opml.org/spec2.opml>
```

## Representation

The `slashpages` well-known resource endpoint provides an [OPML 2.0](http://opml.org/spec2.opml) document. That OPML document's `<body>` is expected to contain one or more `<outline>` elements, referencing some or all of the slash pages of the website, like `/about`, `/now`, `/uses` or `/blogroll`.

Each slash page is an outline of `type="link"`:

- `text` (required): a human readable title of the page.
- `url` (required): the absolute URL of the page. It should be the well-known path on the same site (for example `https://example.com/now/`). If the page lives somewhere else, `url` can point there.
- `category` (recommended): the name of the slash page, in lowercase and with the leading slash. If the page is one of the pages listed on [slashpages.net](https://slashpages.net/), the name should be the one used there (for example `/now`, `/uses` or `/colophon`). Other names are allowed. As defined by OPML 2.0, `category` is a comma-separated list, and every value is one name, so `category="/now,/jetzt"` lists the page under both names. If slashpages.net has more than one name for the same page, the page should be listed under all of them, the first name first (for example `category="/changelog,/log"`). This is only for names of the same page: `/guy` and `/girl` are different pages.

Outlines of `type="include"` pointing to further OPML documents, or any other valid OPML content, are allowed too. Consumers that follow includes should limit how deep they follow them and should not load the same document twice.

Example:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<opml version="2.0">
  <head>
    <title>Slash pages of example.com</title>
    <dateModified>Thu, 01 Oct 2026 12:00:00 GMT</dateModified>
  </head>
  <body>
    <outline type="link" text="About me" category="/about" url="https://example.com/about/" />
    <outline type="link" text="What I am doing now" category="/now" url="https://example.com/now/" />
    <outline type="link" text="Changelog" category="/changelog,/log" url="https://example.com/log/" />
    <outline type="link" text="Blogroll" category="/blogroll" url="https://example.com/blogroll/" />
  </body>
</opml>
```

If in-use by a website, a request to the `slashpages` well-known resource endpoint must return a valid OPML document using the `text/x-opml`, `application/xml`, or `text/xml` MIME type. It should be served with the `Access-Control-Allow-Origin: *` header, so browser based tools can read it.

## As an alternative to `/slashes`

`.well-known/slashpages` supplements but does not replace a human readable `/slashes` page. Specific differences include:

- `.well-known/slashpages` is meant for machines, `/slashes` is meant for people
- `.well-known/slashpages` provides a single location from which a user agent can find all slash pages of a site, without guessing paths like `/now` or `/now/` and without parsing HTML
- `.well-known/slashpages` lists the real URL of every page, also for pages that do not live at their well-known path

## Use Cases

`.well-known/slashpages` benefits:

- Website owners who want to promote their slash pages in a site-wide way
- Directories like [nownownow.com](https://nownownow.com/) or [uses.tech](https://uses.tech/), which want to find and check the `/now` or `/uses` page of a site
- Feed readers and browsers that want to show links to the `/about`, `/contact` or `/blogroll` page of a site a user is reading
- Tools that want to track changes of pages like `/now` without polling a list of guessed URLs

## Security Considerations

- The document should only list pages that are already public.
- A `url` can point to another site. Consumers should not assume that such a page belongs to the site that serves the document, for example a directory should check the page before it lists it.
- Includes can point to other sites and can loop. Consumers should limit how deep they follow them, should not load the same document twice, and should keep in mind that every include is a request to another server.
