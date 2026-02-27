
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

## Nginx:
```conf
server {
    listen 443 ssl;
    server_name aa.bb.xyz app.bb.xyz big.bb.xyz;
    ssl_certificate /usr/local/nginx/ssl/bb.xyz.crt;
    ssl_certificate_key /usr/local/nginx/ssl/bb.xyz.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384';
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:20m;
    ssl_session_timeout 1h;
    ssl_stapling on;
    ssl_stapling_verify on;

    access_log off;
    error_log logs/all.error.log;

    client_max_body_size 10M;
    add_header Access-Control-Allow-Origin "*";
    add_header Access-Control-Allow-Methods "GET,POST,OPTIONS";
    add_header Access-Control-Allow-Headers "Token,Authorization,Origin,X-Requested-With,Content-Type,Accept";  

    location / {
        proxy_pass http://127.0.0.1:26270;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_buffering off;

        proxy_connect_timeout 10s;
        proxy_send_timeout 30s;
        proxy_read_timeout 30s;

        if ($request_method = 'OPTIONS' ) {
            add_header Access-Control-Allow-Origin "*";
            add_header Access-Control-Allow-Methods "GET,POST,OPTIONS";
            add_header Access-Control-Allow-Headers "Token,Authorization,Origin,X-Requested-With,Content-Type,Accept";          
            return 200;
        }
    }
    location /static/ {
        alias /data/project/static/;
        expires 30d;
    }
    location /media/ {
        alias /data/project/media/;
        expires 30d;
    }
    location ~/\. {
        deny all;
    }    
}
```

