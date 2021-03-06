---
title: Discovery
---

When your client talks to an unknown site, you'll need to discover what the
site is capable of and how the site is configured. There are a couple of steps
for this, depending on what you need to discover.


Discovering the API
-------------------

The first step of connecting to a site is finding out whether the site has the
API enabled. Typically, you'll be working with URLs from user input, so the
site you're accessing could be anything. The discovery step lets you verify
the API is available, as well as indicating how to access it.

### Link Header

The preferred way to handle discovery is to send a HEAD request to the
supplied address. The REST API automatically adds a Link header to all
frontend pages that looks like the following:

```
Link: <http://example.com/wp-json/>; rel="https://github.com/WP-API/WP-API"
```

This URL points to the root route (`/`) of the API, which is then used for
further discovery steps.

For sites without "pretty permalinks" enabled, `/wp-json/` isn't automatically
handled by WordPress. This means that normal/default WordPress permalinks will
be used instead. These headers look more like this:

```
Link: <http://example.com/?rest_route=/>; rel="https://github.com/WP-API/WP-API"
```

Clients should keep this variation in mind and ensure that both routes can be
handled seamlessly.

This autodiscovery can be applied to any URL served by a WordPress
installation, so no pre-processing on user input needs to be added. Since this
is a HEAD request, the request should be safe to send blindly to servers
without worrying about causing side-effects.

(Note: The relation (`rel`) is `https://github.com/WP-API/WP-API` during the
plugin's development period. The final version in WordPress core may change
from this.)

### `<link>` Element

For clients with a HTML parser, or running in the browser, the equivalent of
the Link header is included in the `<head>` of frontend pages through a
`<link>` element:

```html
<link rel='https://github.com/WP-API/WP-API' href='http://example.com/wp-json/' />
```

In-browser Javascript can access this via the DOM:

```js
// jQuery method
var $link = jQuery( 'link[rel="https://github.com/WP-API/WP-API"]' );
var api_root = $link.attr( 'href' );

// Native method
var links = document.getElementsByTagName('link');
var link = Array.prototype.filter.apply( links, function ( item ) {
	return ( item.rel === "https://github.com/WP-API/WP-API" );
} );
var api_root = link[0].href;
```

For in-browser clients, or clients without access to HTTP headers, this may be
a more usable way of discovering the API. This is similar to Atom/RSS feed
discovery, so existing code for that purpose may also be automatically
adapted instead.

### RSD (Really Simple Discovery)

For clients with support for XML-RPC discovery, the [RSD method][] may be more
applicable. This is an XML-based discovery format typically used for XML-RPC.
RSD has two steps. The first step is to find the RSD endpoint, supplied as a
`<link>` element:

```html
<link rel="EditURI" type="application/rsd+xml" title="RSD" href="http://example.com/xmlrpc.php?rsd" />
```

[RSD method]: http://cyber.law.harvard.edu/blogs/gems/tech/rsd.html

The second step is to fetch the RSD document and parse the available
endpoints. This involves using an XML parser on a document like the following:

```xml
<?xml version="1.0" encoding="utf-8"?>
<rsd version="1.0" xmlns="http://archipelago.phrasewise.com/rsd">
  <service>
    <engineName>WordPress</engineName>
    <engineLink>http://wordpress.org/</engineLink>
    <homePageLink>http://example.com/</homePageLink>
    <apis>
      <api name="WordPress" blogID="1" preferred="true" apiLink="http://example.com/xmlrpc.php" />
      <!-- ... -->
      <api name="WP-API" blogID="1" preferred="false" apiLink="http://example.com/wp-json/" />
    </apis>
  </service>
</rsd>
```

The REST API always has a `name` attribute with the value equal to `WP-API`.

RSD is the least-preferred method of autodiscovery for a couple of reasons.
The first step of RSD-based discovery involves parsing the HTML to first find
the RSD document itself, which is equivalent to the `<link>` Element
autodiscovery. It then involves another step to parse the RSD document, which
requires a full XML parser.

Where possible, we suggest avoiding RSD-based discovery due to the complexity,
but existing XML-RPC clients may prefer to use this method if they already
have an RSD parser enabled. For XML-RPC clients which wish to use the REST API
as a progressive enhancement to the codebase, this avoids needing to support
different forms of discovery.


Authentication Discovery
------------------------

Discovery is also available for authentication methods available via the API.
The API root's response is an object describing the API, which includes an
`authentication` key:

```json
{
	"name": "Example WordPress Site",
	"description": "YOLO",
	"routes": { ... },
	"authentication": {
		"oauth1": {
			"request": "http://example.com/oauth/request",
			"authorize": "http://example.com/oauth/authorize",
			"access": "http://example.com/oauth/access",
			"version": "0.1"
		}
	}
}
```

The `authentication` value is a map (associative array) of authentication
method ID to authentication options. The options available here are specific
to the authentication method itself. See the [authentication documentation][]
for the options for specific authentication methods.


Extension Discovery
-------------------

The REST API currently doesn't support full extension discovery, but it will
soon. :)
