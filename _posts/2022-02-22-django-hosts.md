
> https://django-hosts.readthedocs.io/en/latest/  
> https://pypi.org/project/django-hosts/


### pip
```bash
# pip install django_hosts
```

### project/project/settings.py
``` python
MIDDLEWARE = [
    "django_hosts.middleware.HostsRequestMiddleware",
    ... ...
    "django_hosts.middleware.HostsResponseMiddleware",
]
ALLOWED_HOSTS = .bb.xyz
CSRF_TRUSTED_HOST = "https://*.bb.xyz,http://*.bb.xyz"
# django hosts
ROOT_HOSTCONF = 'project.hosts'
DEFAULT_HOST = 'inn'

```

### project/project/hosts.py
``` python
from django_hosts import patterns, host

host_patterns = patterns(
    '',
    # aa.bb.xyz
    host(r'aa', 'project.urls', name='aa'),
    # app.bb.xyz
    host(r'app', 'sso.urls', name='app'),
    # big.bb.xyz
    host(r'big', 'big.urls', name='big'),
)

```

### project/project/url.py
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path("admin/", admin.site.urls),
    path('mixed/', include('mixed.urls')),
]

```

### app/url.py
```
from django.urls import path
from . import views

urlpatterns = [
    path("", views.app_index, name='app_index'),
    path(
        'app_check',
        views.app_check,
        name='app_check',
    )
]
```

### big/url.py
```
from django.urls import path
from . import views

urlpatterns = [
    path("", views.big_index, name='big_index'),
    path(
        'big_check',
        views.big_check,
        name='big_check',
    )
]
```
