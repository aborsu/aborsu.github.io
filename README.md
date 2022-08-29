# aborsu.github.io

## Django + Domino Datalab

1. Developping

Running django's runserver to work on django.

https://docs.dominodatalab.com/en/5.2/user_guide/2c01ae/run-multiple-applications-in-a-workspace/

### SCRIPT_NAME and Django
https://stackoverflow.com/questions/33051034/django-not-taking-script-name-header-from-nginx#:~:text=Django%20expects%20its%20requests%20to%20start%20with%20the,will%20not%20prepend%20it%20with%20your%20script%20name.

Not working with developement server (runserver), as all headers with underscores are removed.
https://github.com/django/django/blob/a9e7beb959bc726eab1c192d2625d6ff6cfa70f4/django/core/servers/basehttp.py#L189

Apparently Flask/[Werkzeug use X_FORWARDED_PREFIX](https://werkzeug.palletsprojects.com/en/2.0.x/middleware/proxy_fix/) which is set from the corresponding http header and is then passed as SCRIPT_NAME to the rest of the application. Unfortuanately this is not supported by gunicorn and django... but with a little hack

```python3
# app/wsgi.py
from django.core.wsgi import get_wsgi_application

_application = get_wsgi_application()

def application(environ, start_response):
    script_name = environ.get('X_FORWARDED_PREFIX', '')
    if script_name:
        environ['SCRIPT_NAME'] = script_name
        path_info = environ['PATH_INFO']
        if path_info.startswith(script_name):
            environ['PATH_INFO'] = path_info[len(script_name):]

    scheme = environ.get('HTTP_X_SCHEME', '')
    if scheme:
        environ['wsgi.url_scheme'] = scheme
    return _application(environ, start_response)
```
