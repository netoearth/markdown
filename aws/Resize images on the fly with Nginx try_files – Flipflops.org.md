Nginx is fantastic as a high performance server that can serve static files very, very quickly with minimal, predictable use of resources. Static files are the CSS, Javascript and boilerplate images that make up the design of a website. As the name suggests, static files tend not to change very often. These days the chances are that they have already been optimised, minified and compressed before being deployed to the web server.

Nginx doesn’t handle requests for scripts like PHP or Python directly, but instead acts a reverse proxy and will pass requests to backend server. We tend to use php-fpm but the backend can be anything, even Apache.

Websites typically have a lot of images uploaded by users that are needed in lots of different variants (thumbnail, crop, large etc.). These user generated images obviously can’t be optimised and resized ahead of time like all the boilerplate images.

One solution is to generate all the different image variants when an image is uploaded, but this isn’t very flexible, can create a noticeable delay as all the variants are generated, and things can get complicated if you need to create a new variant for thousands and thousands of images.

There are lots of solutions that will resize images dynamically but these tend to send all requests for images to a script that will serve existing images (hopefully from a cache) or attempt to generate a new image on the fly, serve the new image and save it in a cache for future use. This approach is tried and tested but it is unnecessary overhead calling a script for every image request. It is much more efficient to serve images straight from the disk, something Nginx excels at, and not have to involve a script at all.

A nice solution is to use the Nginx [try\_files](https://nginx.org/en/docs/http/ngx_http_core_module.html#try_files) directive to serve an image directly, if it exists and only send the request to the backend if the image (variant) requested doesn’t exist.

From the [documentation for **try\_files**](https://nginx.org/en/docs/http/ngx_http_core_module.html#try_files):

> Checks the existence of files in the specified order and uses the first found file for request processing; the processing is performed in the current context. The path to a file is constructed from the file parameter according to the root and alias directives. It is possible to check directory’s existence by specifying a slash at the end of a name, e.g. “$uri/”. If none of the files were found, an internal redirect to the uri specified in the last parameter is made. For example:
> 
> ```
> location /images/ {
>     try_files $uri /images/default.gif;
> }
> 
> location = /images/default.gif {
>     expires 30s;
> }
> 
> ```

In the above example if the specified image is missing, /images/default.gif is returned instead and has it’s ‘Expires’ header set to 30 seconds.

## Simplified Nginx conf file

See how we can use this in practice

```
server {
    listen 80;

    access_log /var/log/nginx/example.com.access.log main;
    error_log /var/log/nginx/example.com.error.log;
    root /var/www/vhosts/example.com/htdocs;
    index index.php index.html;
    server_name example.com;

    # Check if the requested image exists and if not pass to @backend
    location ~* ^.+.(jpg|jpeg|png|gif)$ {
        try_files $uri @backend;
        add_header Pragma "public";
        add_header Cache-Control "public";
        expires     1y;
        access_log  off;
        log_not_found off;
     }



    # Other static files
    location ~* ^.+.(ico|css|zip|tgz|gz|rar|bz2|doc|xls|exe|pdf|ppt|txt|tar|mid|midi|wav|bmp|rtf|js|eot|woff|svg|htc)$ {
        add_header Pragma "public";
        add_header Cache-Control "public";
        expires     1y;
        access_log  off;
        log_not_found off;
    }

    location / {
        # Check if a file exists, or route it to index.php.
        # Configure depending on application used
        try_files $uri $uri/ /index.php?$query_string;
    }

    location @backend {
        rewrite ^(.*)$ /images.php?url=$uri;
    }

    location ~\.php$ {
        try_files $uri =404;
        fastcgi_pass unix:/dev/shm/apache-php.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include /etc/nginx/fastcgi_params;
    }

}

```

In the solution above @backend passes requests to /images.php. Images.php bypasses the rest of the application and is simply contains code to necessary resize images on the fly. Write your own or make use of an existing library. Personally, I like [Intervention Image](http://image.intervention.io/).