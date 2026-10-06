# 🚀 Django-Next.js

Seamlessly bridge **Django** and **Next.js** into a single, unified application. 

When a user visits your site, Django intercepts the initial request, fetches the HTML response directly from your Next.js server, and delivers it. Afterward, seamless client-side routing takes over via Next.js internal paths (`/_next/...`)—all without extra round-trips to Django!

---

## ✨ Why Choose Django-Next.js?

* **Gradual Migrations:** Perfect for adding Next.js features or pages incrementally to a legacy Django monolith.
* **Streamlined Dev Experience:** No complex Nginx reverse proxy required during development. Everything runs smoothly on a single port with full **Fast Refresh** support.
* **Production Ready:** Simplifies production proxying by eliminating the need to hardcode a growing list of Next.js page paths.

---

## 📦 Quick Start

### 1. Installation
```bash
pip install django-nextjs
```

### 2. Configure Django Settings
Add the app to your `INSTALLED_APPS`:
```python
INSTALLED_APPS = [
    ...
    "django_nextjs",
]
```

### 3. Setup ASGI Middleware
Enable the middleware in your `asgi.py` to support asset routing and live-reload WebSockets:
```python
import os
from django.core.asgi import get_asgi_application

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "myproject.settings")
django_asgi_app = get_asgi_application()

from django_nextjs.asgi import NextJsMiddleware

application = NextJsMiddleware(django_asgi_app)
```

### 4. Wire Up Your Views
Map your Next.js pages directly inside your Django `urls.py`:
```python
from django_nextjs.views import nextjs_page

urlpatterns = [
    path("my/page", nextjs_page(), name="my_page"),
    # Recommended for Next.js App Router streaming:
    path("other/page", nextjs_page(stream=True), name="other_page"),
]
```

---

## ⚙️ Essential Configuration

Configure default behaviors by adding `NEXTJS_SETTINGS` to your Django settings file:

```python
NEXTJS_SETTINGS = {
    "nextjs_server_url": "http://127.0.0.1:3000",
    "ensure_csrf_token": True,
    "public_subdirectory": "/next",
}
```

* **`nextjs_server_url`**: The active endpoint of your running Next.js instance.
* **`ensure_csrf_token`**: Automatically generates and seeds a CSRF token on the initial request—ideal for data fetching via POST/GraphQL inside server-side methods.
* **`public_subdirectory`**: Subfolder inside the Next.js `public` directory used for static assets (default: `/next`).

---

## 🌐 Production Nginx Setup

Route traffic efficiently in production:

```conf
location /_next/ {
    proxy_pass http://127.0.0.1:3000;
    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}

location /next/ {
    alias NEXTJS_PATH/public/next/;
    add_header Cache-Control "public, max-age=0";
}
```

---

## 💡 Quick Tips & Notes

* **ASGI Required:** An ASGI server (like **Daphne** or **Uvicorn**) is required for Fast Refresh to work.
* **Trailing Slashes:** Set `APPEND_SLASH = False` in your Django settings to prevent redirect loops.
* **Run Separately:** Remember to spin up your Next.js development server (`npm run dev`) alongside Django!

---

## 🤝 Contributing & License

We love community contributions! To get started:
1. Install dev dependencies: `pip install -e '.[dev]'`
2. Set up hooks: `pre-commit install`
3. Submit your pull request!

Distributed under the **MIT License**. Star 🌟 on GitHub if you love this project!