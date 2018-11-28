name: inverse
layout: true
class: center, middle, inverse
---
template: inverse

# Django

The Web framework for perfectionists with deadlines

---
layout: false

## 1 + 1 == 3

```python
import ctypes

ctypes.c_int8.from_address(id(2)+16).value=3
```

```python
import sys
import ctypes

ctypes.memmove(id(2), id(3), sys.getsizeof(1))
```

```python
import ctypes

ctypes.memset(id(2)+16, 3, 1)
```

---

## 目录

* 概念介绍
    * Object-relational mapper
    * URLs and views
    * Templates
    * Forms
    * Authentication
    * Admin
* 项目实例

---

## Object-relational mapper

* 使用 python 定义数据类型
* 统一的数据库访问 API
* 对接不同的数据库 (sqlite/mysql/postgresql 等)

.left-column-50[
  ![left](/assets/2016-5-19-django-img/image1.png)
]
.right-column-50[
  ![right](/assets/2016-5-19-django-img/image2.png)
]

---

## URLs and views

To design URLs for an application, you create a Python module called a URLconf.
Like a table of contents for your app, it contains a simple mapping between URL
patterns and your views.

```python
from django.conf.urls import url
from . import views


urlpatterns = [
    url(r'^bands/$', views.band_listing, name='band-list'),
    url(r'^bands/(\d+)/$', views.band_detail, name='band-detail'),
    url(r'^bands/search/$', views.band_search, name='band-search'),
]
```

```python
from django.shortcuts import render


def band_listing(request):
    """A view of all bands."""
    bands = models.Band.objects.all()
    return render(request, 'bands/band_listing.html', {'bands': bands})
```

---

## Templates

Django's template language is designed to strike a balance between power and ease.
It's designed to feel comfortable and easy-to-learn to those used to working with
HTML, like designers and front-end developers. But it is also flexible and highly
extensible, allowing developers to augment the template language as needed.

```xml
<html>
  <head>
    <title>Band Listing</title>
  </head>
  <body>
    <h1>All Bands</h1>
    <ul>
    {% for band in bands %}
      <li>
        <h2><a href="{{ band.get_absolute_url }}">{{ band.name }}</a></h2>
        {% if band.can_rock %}<p>This band can rock!</p>{% endif %}
      </li>
    {% endfor %}
    </ul>
  </body>
</html>
```
---

## Forms

Django provides a powerful form library that handles rendering forms as HTML,
validating user-submitted data, and converting that data to native Python types.
Django also provides a way to generate forms from your existing models and use
those forms to create and update data.

```python
from django import forms


class BandContactForm(forms.form):
    subject = forms.CharField(max_length=100)
    message = forms.CharField()
    sender = forms.EmailField()
    cc_myself = forms.BooleanField(required=False)
```

---

## Authentication

Django comes with a full-featured and secure authentication system. It handles
user accounts, groups, permissions and cookie-based user sessions. This lets
you easily build sites that let users to create accounts and safely log in/out.

```python
from django.contrib.auth.decorators import login_required
from django.shortcuts import render


@login_required
def my_protected_view(request):
    """A view that can only be accessed by logged-in users"""
    return render(request, 'protected.html', {'current_user': request.user})
```

---

## Admin

One of the most powerful parts of Django is its automatic admin interface.
It reads metadata in your models to provide a powerful and production-ready
interface that content producers can immediately user to start managing content
on your site. It's easy to set up and provides many hooks for customization.

```python
from django.contrib import admin
from bands.models import Band, Member


class MemberAdmin(admin.ModelAdmin):
    """Customize the look of the auto-generated admin for the Member model"""
    list_display = ('name', 'instrument')
    list_filter = ('band', )

admin.site.register(Band)  # Use the default options
admin.site.register(Member, MemberAdmin)  # Use the customized options
```

---

## 项目实例

.left-column[
  ![left](/assets/2016-5-19-django-img/image3.png)
]

.right-column[
**操作步骤**

* pip install django==1.9.0
* django-admin startproject test_site
* 进入项目根目录(有 `manage.py` 的那个目录)
* python manage.py startapp knowledge
]

---
template: inverse

## Thank you!

2016-5-19
