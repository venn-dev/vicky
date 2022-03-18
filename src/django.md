# Django

## Static files

Django is too cool to serve static files.

We need either something like
[whitenoise](https://whitenoise.evans.io/en/stable/) which
[in the background](https://whitenoise.evans.io/en/stable/#isn-t-serving-static-files-from-python-horribly-inefficient)
uses the kernel's `sendfile` syscall.

Or, we can serve them above Django, from our web server / reverse proxy,
if we have one (eg. nginx). See [nginx](nginx.md) for related configuration
examples.

In both cases, one needs to have these two lines in their `settings.py`:

```
STATIC_URL = "/static/"
STATIC_ROOT = os.path.join(BASE_DIR, "static")
```

`STATIC_URL` is included in the Django project generation. `STATIC_ROOT` needs
to be added manually.

Django docs how-to guide on static files
[here](https://docs.djangoproject.com/en/dev/howto/static-files/) and
[here](https://docs.djangoproject.com/en/dev/ref/contrib/staticfiles/).

Also, enabling manifest static files is usually a good idea for high-quality
cache busting. To do this, add this line as well in your `settings.py`.

```
STATICFILES_STORAGE = "django.contrib.staticfiles.storage.ManifestStaticFilesStorage"
```
