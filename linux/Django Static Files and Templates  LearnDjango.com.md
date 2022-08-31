-   By Will Vincent
-   May 5, 2022
-   [14 Comments](https://learndjango.com/tutorials/django-static-files#disqus_thread)

Static files like CSS, JavaScript, and fonts are a core piece of any modern web application. They are also typically confusing for Django newcomers since Django provides tremendous flexibility around _how_ these files are used. This tutorial will demonstrate current best practices for configuring static files in both local development and production.

## Local Development

For local development the Django web server automatically serves static files and minimal configuration is required. A single Django project often contains multiple apps and by default Django will look within each app for a `static` directory containing static files.

For example, if one of your apps was called `blog` and you wanted to add a CSS file to it called `base.css`, you would need to first create a new directory called `blog/static` and then add the file within it at the location `blog/static/style.css`.

However, since most Django projects involve multiple apps and shared static files, it is common on larger projects to instead create a base-level folder typically named `static`. This can be added from the command line with the `mkdir` command:

For demonstration purposes let's also add a `base.css` file. Assuming you had only just started a new Django project with the `startproject` command your directory structure would now look like this:

```
├── django_project
│   ├── __init__.py
|   ├── asgi.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── manage.py
├── static
│   ├── base.css

```

Within the `settings.py` file, near the bottom, there is a single line of configuration for static files called [STATIC\_URL](https://docs.djangoproject.com/en/dev/ref/settings/#std:setting-STATIC_URL), which is the location of static files in our project.

```
# django_project/settings.py
STATIC_URL = "/static/"

```

This means that all static files will be stored in the location `http://127.0.0.1:8000/static/` or `http://localhost:8000/static/`. And if you wanted to access the `base.css` file its location would be `http://127.0.0.1:8000/static/base.css` or `http://localhost:8000/static/base.css`.

## Loading static files into Templates

Loading static files in a template is a two-step process:

-   add `{% load static %}` at the top of the template
-   add the [{% static %}](https://docs.djangoproject.com/en/dev/ref/templates/builtins/#std:templatetag-static) template tag with the proper link

There are two main ways to structure templates in a Django project as [outlined in this tutorial](https://learndjango.com/tutorials/template-structure). Let's assume we are using a `templates/base.html` file for a Blog project. To add our static `base.css` file to it we'd start by adding `{% load static %}` at the top of the file and then use the `{% static %}` tag with the path to it. Since the `STATIC_URL` is already set to `/static/` we don't need to write the full route out of `static/base.css` and can instead shorten it to `base.css`.

```
<!-- templates/base.html -->
{% load static %}
<!DOCTYPE html>
<html>
<head>
  <title>Learn Django</title>
  <link rel="stylesheet" href="{% static 'base.css' %}">
</head>
...
</html>

```

If you save and reload the web page, you'll see the changes. Adding links for either images in an `img` folder or JavaScript in a `js` folder would look as follows:

```
<img src="{% static 'img/path_to_img' %}">
<script src="{% static 'js/base.js' }"></script>

```

## collectstatic

Serving static files in production requires several additional steps and is where the confusion typically arises for Django newcomers. Local development is designed to keep things nice and easy, but it is far from performant. In a production environment it is more efficient to combine **all** static files in the Django project into one location and serve that a single time.

Django comes with a built-in management command, [collectstatic](https://docs.djangoproject.com/en/dev/ref/contrib/staticfiles/#django-admin-collectstatic), that does this for us.

We need three more configurations in our `settings.py` file though before `collectstatic` will work properly. The first is [STATICFILES\_DIRS](https://docs.djangoproject.com/en/dev/ref/settings/#std:setting-STATICFILES_DIRS) which defines additional locations, if any, the staticfiles app should look within when running `collectstatic`. In our simple example the only location for local files is the `static` directory so we will set that now.

```
# django_project/settings.py
STATIC_URL = "/static/"
STATICFILES_DIRS = [BASE_DIR / "static"]  # new

```

Next up is [STATIC\_ROOT](https://docs.djangoproject.com/en/dev/ref/settings/#static-root) which sets the absolute location of these collected files, typically called `staticfiles`. In other words, when `collecstatic` is run locally it will combine all available static files, as defined by `STATICFILES_DIRS` and place them within a directory called `staticfiles`. This is how we set that configuration.

```
# django_project/settings.py
STATIC_URL = "/static/"
STATICFILES_DIRS = [BASE_DIR / "static"]  
STATIC_ROOT = STATIC_ROOT = BASE_DIR / "staticfiles"  # new

```

The final step is [STATICFILES\_STORAGE](https://docs.djangoproject.com/en/dev/ref/settings/#staticfiles-storage), which is the _file storage engine_ used when collecting static files with the `collectstatic` command. By default, it is implicitly set to `django.contrib.staticfiles.storage.StaticFilesStorage`. Let's make that explicit for now in our `django_project/settings.py` file.

```
# django_project/settings.py
STATIC_URL = "/static/"
STATICFILES_DIRS = [BASE_DIR / "static"]  
STATIC_ROOT = STATIC_ROOT = BASE_DIR / "staticfiles" 
STATICFILES_STORAGE = "django.contrib.staticfiles.storage.StaticFilesStorage"  # new

```

Now we can run the command `python manage.py collectstatic` which will create a new `staticfiles` directory.

```
(.venv) $ python manage.py collectstatic

```

If you look within it you'll see that `staticfiles` also has folders for `admin` (the built-in admin has its own static files), `staticfiles.json`, and whatever directories are in your `static` folder.

If you now add a new static file to the `static` directory it will immediately be available for local usage. It is only for production where the file won't be present unless you run `python manage.py collectstatic` each and every time. For this reason, running `collectstatic` is typically added to deployment pipelines and is done by default on Heroku.

As a brief recap:

-   `STATIC_URL` is the _URL location_ of static files located in `STATIC_ROOT`
-   `STATICFILES_DIRS` tells Django _where to look_ for static files in a Django project, such as a top-level `static` folder
-   `STATIC_ROOT` is the _folder location_ of static files when `collectstatic` is run
-   `STATICFILES_STORAGE` is the _file storage engine_ used when collecting static files with the `collectstatic` command.

## Production

Even though we've configured our Django project to collect static files properly, there's one more step involved which is not included in the official Django docs. That is the configuration of [WhiteNoise](http://whitenoise.evans.io/en/stable/) to serve the static files in production. The first step is to install the latest version of the package:

```
(.venv) $ python -m pip install whitenoise==6.0.0

```

Then in the `django_project/settings.py` file make three changes:

-   add `whitenoise` to the `INSTALLED_APPS` **above** the built-in `staticfiles` app
-   under `MIDDLEWARE`, add a new `WhiteNoiseMiddleware` on the third line
-   change `STATICFILES_STORAGE` to use `WhiteNoise`.

It should look like the following:

```
# django_project/settings.py
INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "whitenoise.runserver_nostatic",  # new
    "django.contrib.staticfiles",
]

MIDDLEWARE = [
    "django.middleware.security.SecurityMiddleware",
    "django.contrib.sessions.middleware.SessionMiddleware",
    "whitenoise.middleware.WhiteNoiseMiddleware",  # new
    "django.middleware.common.CommonMiddleware",
    "django.middleware.csrf.CsrfViewMiddleware",
    "django.contrib.auth.middleware.AuthenticationMiddleware",
    "django.contrib.messages.middleware.MessageMiddleware",
    "django.middleware.clickjacking.XFrameOptionsMiddleware",
]

...

STATIC_URL = "/static/"
STATICFILES_DIRS = [BASE_DIR / "static"]  
STATIC_ROOT = STATIC_ROOT = BASE_DIR / "staticfiles"
STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'  # new

```

That's it! Run `python manage.py collectstatic` again so that the files are stored using WhiteNoise. And then deploy with confidence to the hosting platform of your choice.

## CDNs

Content Delivery Networks (CDNs) are useful on very high traffic sites where performance is a concern. Rather than serving static files from your Django server you post them on a dedicated CDN network and then call them. The [official WhiteNoise documentation](https://whitenoise.evans.io/en/stable/django.html#use-a-content-delivery-network) has additional instructions on this step.

## Conclusion

Configuring static files is a core part of any Django project. If you'd like to see multiple examples of real-world Django projects alongside detailed explanation, check out my book [Django For Beginners](https://djangoforbeginners.com/). The first several chapters can be read for free online.