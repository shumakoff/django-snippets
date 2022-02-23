## How to disable column sorting in admin for Django <= 1.10

In `admin.py`:

```
from django.contrib.admin.views.main import ChangeList

class UnorderedChangeList(ChangeList):

    def get_ordering_field(self, field_name):
        return None
```
in `get_ordering_field` you can filter, which fields are allowed for sorting.

Now override method `get_changelist` of model admin:

```
    def get_changelist(self, request, **kwargs):
        return UnorderedChangeList
```

Now user sorting is disabled on back end of admin site, but user can still try to sort on front end. It won't work, but
to avoid any unnecessary tickets let's disable front end part too.

Make custom admin template for your model and put it in 

`<template_dir>/admin/<app_name>/<model_name>`

template name should be `change_list.html`

In `change_list.html`:

```
{% block extrahead %}
  {{ block.super }}
  <script type="text/javascript">
    django.jQuery(function ($) {
      $(".sortable").each(function (index) {
        $(this).find(".text").css("padding", "2px 5px");
        $(this).find("a").contents().unwrap();
      })
    })
  </script>
{% endblock %}
```
