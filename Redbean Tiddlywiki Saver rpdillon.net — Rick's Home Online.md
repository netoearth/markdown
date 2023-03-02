[Tiddlywiki](https://tiddlywiki.com/) is a great tool, but because it runs in the browser, it has only limited access to the filesystem to save changes. This has led to dozens of tools called "savers" that work with Tiddlywiki to save it. These span the gamut, including full fledged desktop apps (most notably [TiddlyDesktop](https://tiddlywiki.com/static/TiddlyDesktop.html)), browser extensions, and server applications of varying complexity. But all of these rely on a single capability built-in to Tiddlywiki that allows it to save itself to any WebDAV server. In addition to purpose-built servers, Tiddlywiki can save itself to WebDAV servers available for:

-   [Nginx](https://nginx.org/en/docs/http/ngx_http_dav_module.html)
-   [Apache](https://httpd.apache.org/docs/2.4/mod/mod_dav.html)
-   [Caddy](https://caddyserver.com/docs/modules/http.handlers.webdav)

I started to consider writing a tiny version of a WebDAV tiddlywiki saver after I ran across jart's phenomenal [Redbean](https://redbean.dev/) project, which is a webserver contained as an αcτµαlly pδrταblε εxεcµταblε. This approach appeals strongly to my love of small, self-contained tools (one of the reasons I love Tiddlywiki!) It's a web server application that is also a zip archive. By adding Lua code and assets to the zip archive, and then executing it on most any x86\_64 system, it provides a self-contained, offline web application. After reading through the documentation for Redbean, it occurred to me that it should be not only possible, but quite easy to write a Redbean application that not only is a saver for Tiddlywiki, but can bundle the wiki into the application itself, creating a completely self-contained Tiddlywiki application.

But, I've never worked with Redbean before, so these are my notes on implementing this idea.

## Redbean's Hello World

Here's my hello world in `index.lua`:

```
Write("<html>")
Write("<head><title>Redbean Lua Demo</title></head>")
Write("<body>")
Write("This is just a demo page...hello, world!")
Write("</body>")
Write("</html>")
```

I then build the executable using a small script:

```
if [! -f redbean.com ]; then
    curl https://redbean.dev/redbean-latest.com > redbean.com
    chmod +x redbean.com
fi
cp redbean.com wiki.com
zip wiki.com index.lua
./wiki.com
```

## Serving a Tiddlywiki

Instead of generating HTML in the Lua code, we already have a bundle of HTML to serve: a Tiddlywiki! This means all the Lua code has to do is serve that asset:

```
ServeAsset("wiki.html")
```

Of course, we need to provide that `wiki.html` file to serve. Let's grab one:

```
curl https://tiddlywiki.com/empty.html > wiki.html
zip wiki.com index.lua wiki.html
./wiki.com
```

Opening `localhost:8080` in the browser, we can see that the wiki loads, and can be edited, but there's no saving yet.

## Adding Saving

Redbean has a surprisingly comprehensive set of methods available to tackle the saving piece. In order to enable saving for Tiddlywiki with the built-in DAV tooling, the server needs to understand four major HTTP methods: `GET`, `HEAD`, `OPTIONS`, and `PUT`.

To implement `HEAD`, we want to provide two relevant headers: the `Content-Type` and the `Content-Length`. Right now, the goal is to have Redbean store exactly one wiki, so the path to the resource doesn't matter. But since we'll have to dispatch on method type, we need to extract that first. At the same time, we'll hardcode the path to the wiki inside the zip:

```
method = GetMethod()
WIKI_PATH="wiki.html"
```

To implement `GET` we simply use the logic from above:

```
if method == "GET" then
   ServeAsset(WIKI_PATH)
end
```

For `HEAD`, two headers are required, and they can be set with `SetHeader`. One is `Content-Length`, which is dynamic, so it first needs to load the wiki and get its length before setting the headers:

```
-- GET method above
elseif method == "HEAD" then
   wiki = LoadAsset(WIKI_PATH)
   SetStatus(200)
   SetHeader("Content-Type", "text/html")
   SetHeader("Content-Length", tostring(#wiki))
end
```

Implementing `OPTIONS` has to emulate WebDAV just enough to trick Tiddlywiki into saving using that mechanism. I based these headers from the very sleek [Ruby saver](https://gist.github.com/jimfoltz/ee791c1bdd30ce137bc23cce826096da) written by Jim Foltz.

```
-- GET and HEAD methods above
elseif method == "OPTIONS" then
   SetStatus(200)
   SetHeader("allow", "GET,HEAD,POST,OPTIONS,CONNECT,PUT,DAV,dav")
   SetHeader("x-api-access-type", "file")
   SetHeader("dav", "tw5/put")
end
```

The last method is `PUT`, which needs to write the payload back to the Redbean archive. Luckily, Redbean provides a `StoreAsset` function for exactly this case:

```
-- GET, HEAD, and OPTIONS methods above
elseif method == "PUT" then
   length = tonumber(GetHeader("Content-Length"))
   body = GetBody()
   StoreAsset(WIKI_PATH, body)
   SetStatus(200)
end
```

## Putting It All Together

So, the full script is a compact 21 lines of Lua:

```
method = GetMethod()
WIKI_PATH="wiki.html"

if method == "GET" then
   ServeAsset(WIKI_PATH)
elseif method == "HEAD" then
   wiki = LoadAsset(WIKI_PATH)
   SetStatus(200)
   SetHeader("Content-Type", "text/html")
   SetHeader("Content-Length", tostring(#wiki))
elseif method == "OPTIONS" then
   SetStatus(200)
   SetHeader("allow", "GET,HEAD,POST,OPTIONS,CONNECT,PUT,DAV,dav")
   SetHeader("x-api-access-type", "file")
   SetHeader("dav", "tw5/put")   
elseif method == "PUT" then
   length = tonumber(GetHeader("Content-Length"))
   body = GetBody()
   StoreAsset(WIKI_PATH, body)
   SetStatus(200)
end
```

Then all that's needed is to inject `index.lua`, along with a wiki, into the archive and run it:

```
zip wiki.com wiki.html index.lua
./wiki.com
```

While this loads and allows editing as before, upon saving, Tiddlywiki throws an error, and the Redbean logs reveal a `413` error: the payload during the `PUT` is too large. This is because a Tiddlywiki is over 1MB, but the maximum payload size of Redbean is 65k. This can be adjusted during launch...here we'll use 5MB:

```
./wiki.com -M 5000000
```

This approach allows Tiddlywiki to load and save exactly as expected, and packs everything into a single 2.5M file.

## Changelog

-   2022-11-19 - Added details about WebDAV saving mechanism after [feedback on HN](https://news.ycombinator.com/item?id=33673116).