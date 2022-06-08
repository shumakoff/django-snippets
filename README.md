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

## Admin date/datetime widget time format when clicking on presets like "now"

in template:

```
  <script type="text/javascript">
    window.addEventListener('load', function () {
      DateTimeShortcuts['handleClockQuicklink'] = function (num, val) {
        var d;
        if (val == -1) {
          d = DateTimeShortcuts.now();
        } else {
          d = new Date(1970, 1, 1, val, 0, 0, 0)
        }
        DateTimeShortcuts.clockInputs[num].value = d.strftime('%H:%M');
        DateTimeShortcuts.clockInputs[num].focus();
        DateTimeShortcuts.dismissClock(num);
      }
    });
  </script>
```
in form:

`widget=AdminTimeWidget(format='%H:%M'), input_formats=['%H:%M']`
changes format to '%H:%M' instead of default '%H:%M:%S'

## Add ModelTranslation translation for existing models

When you enable ModelTranslation for a model with existing data things will brake [ModelTranslation](https://django-modeltranslation.readthedocs.io/en/latest/commands.html#the-update-translation-fields-command)

If for some reason you can't use this command, then make custom data migration like this. Make sure to change `field_fallback_lang` to an actual field name like `title_ka`.
```
from __future__ import unicode_literals

from django.db import migrations
from django.db.models import F


def fill_translation(apps, schema_editor):
    model_to_translate = apps.get_model('app', 'MyModel')
    model_to_translate.objects.filter(field_fallback_lang__isnull=True).update(field_fallback_lang=F('name'))


class Migration(migrations.Migration):

    dependencies = [
        ('app', '0001_initial'),
    ]

    operations = [
        migrations.RunPython(fill_translation),
    ]
```
