In some organizations, you may have seen “short links” like

`http://go/foo-bar`

or

`http://x/12345`.

These are short for (and resolve to) something else, like `https://example.com/go/foo-bar` or `https://example.com/x/reports/12345`.

One way to set this up is to use a proxy and something called [Proxy Auto Configuration (PAC)](https://en.wikipedia.org/wiki/Proxy_auto-config). A PAC file is a small snippet of Javascript that directs browsers to use a proxy for certain hostnames. For more details, see [MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Proxy_servers_and_tunneling/Proxy_Auto-Configuration_PAC_file).

Here’s a simple setup for macOS using the [Caddy](https://caddyserver.com/) proxy that will let you type `http://g/foo bar` and it will take you to `http://google.com/search?q=foo+bar`. (This example is a bit of a contrived, since all browsers today support querying a search engine in the URL bar, but it gets the idea across).

Install Caddy:

In a new file `proxyconfig.pac`:

```
function FindProxyForURL(url, host) {
  if (host == "g") {
    return "PROXY localhost:9999";
  } else {
    return "DIRECT";
  }
}
```

For details on how the PAC file works, see the aforementioned MDN page.

Next, we’ll configure Caddy to (a) serve a static PAC file, and (b) proxy `http://g` requests to Google. In a new file `caddyfile` in the same directory:

```
# Serve up the PAC file.
http://localhost:9999 {
    log
    handle /proxyconfig.pac {
        file_server {
        }
    }
}
# Rewrite http://g requests.
http://g:9999 {
    log
    @mymatcher {
        path_regexp /(.*)
    }
    redir @mymatcher http://www.google.com/search?q={http.regexp.1}
}
# Redirect requests for Google.
http://, https:// {
    redir {scheme}{hostport}{uri}
}
```

(I’m new to Caddy so the above may not be good form, secure, etc. You’ve been warned.)

Start Caddy:

Install the PAC file on macOS by opening System Preferences, go to Network > Advanced > Proxies > Automatic Proxy Configuration. Set the Proxy Configuration File URL to `http://localhost:9999/proxyconfig.pac`.<sup id="fnref:1"><a href="https://kalaracey.github.io/how-short-links-work/#fn:1" role="doc-noteref">1</a></sup>

Now, you should be able to type in `g/foo bar` into your browser, hit enter, and it should direct you to Google’s search results for “foo bar”. At least in Chrome, when you type it in, there may be two results in the drop-down suggestion list - you need to make sure to select the blue one (you should only need to do this once).

If you modify the PAC file, your browser may not immediately pick the change up. To force Chrome to do so, go to [chrome://net-internals#proxy](chrome://net-internals/#proxy) and click ‘Re-apply settings’.

___

1.  `file:///path/to/proxyconfig.pac` does not seem to work ([https://superuser.com/a/565071)](https://superuser.com/a/565071)). [↩︎](https://kalaracey.github.io/how-short-links-work/#fnref:1)