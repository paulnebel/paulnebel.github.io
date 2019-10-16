---
title: "Overriding Django Widget Templates"
meta: "A simple method for overriding widget templates in Django. Specifically, this describes how to override the RelatedFieldWidgetWrapper for ForeignKey and ManyToMany fields."
categories: Django
slug: "overriding-django-widget-templates"
---

# Overriding Django Widget Templates
I spent all day yesterday trying to find a quick and easy way to override the widget wrapper for a ForeignKey field in a Django Admin form.  Despite too many internet searches to mention I couldn't find a quick, easy and comprehensible way of achieving my goal.  I eventually found a way to override the widget template by combining everything I'd read and sleeping on it.  This post is the result.

## Aim
I have a model which links to itself via a parent-child relationship.  I want to add properties to this relationship so I have created an [intermediate model](https://docs.djangoproject.com/en/2.2/topics/db/models/#extra-fields-on-many-to-many-relationships) to maintain links between parents and children using [ForeignKeys](https://docs.djangoproject.com/en/2.2/ref/models/fields/#foreignkey).  When editing a parent, I display the associated children using a [TabularInline]() admin model.

The default widget for **ForeignKey** fields within a TabularInline is the [RelatedFieldWidgetWrapper](https://github.com/django/django/blob/8f6860863e34cb1cbe24161f1c4e7c79007e93dc/django/contrib/admin/widgets.py).  This widget is almost what I need, but not quite.  The issue for me is that the default behaviour for creating new children or editing existing children is to open a popup.  Since the model being created/edited is identical to the parent model, I could easily find myself in a very confusing loop whereby I create/edit a child in a popup and then create/edit another child for that child in another popup... etcetera, ad infinitum.

I want to to avoid this by updating the URLs associated with the add/edit actions.  This update will replace the change page for the parent model currently being edited with a change page for the new/updated child model, rather than opening the new/updated child page as a popup over the parent model.

I'll be adding a breadcrumb at the top of these change pages so that I can easily navigate back to the parent and will be auto-saving parents and children at appropriate points in the process.  The details of the breadcrumbs and model lifecycle are not the topic of this post.  Overriding the links in the default widget *is*.

## Method
All I need to do is to override the default widget template html.  The interwebs appears not to contain any *comprehensible* information on how to do this.  [StackOverflow](https://stackoverflow.com/) contains lots of answers to this particular question, but these are mostly obscure, over-complicated or simply lacking in sufficient detail for those of us who are not already Django experts.  Here is my humble contribution.

### Examine the implementation
The [Django documentation](https://docs.djangoproject.com) is extremely detailed but sometimes a little short on examples.  This sometimes means that you have to trawl through many pages in order to piece the answer together.  The key to the current problem came from looking at [built-in widgets](https://docs.djangoproject.com/en/2.2/ref/forms/widgets/#built-in-widgets) and [how to override](https://docs.djangoproject.com/en/2.2/ref/forms/renderers/#overriding-built-in-widget-templates) them, in combination with [this snippet](https://djangosnippets.org/snippets/2565/).

It seems that if I create my own widget that inherits from the **RelatedFieldWidgetWrapper** I can then define the template it uses via the `template_name` attribute.  This allows me to retain the default functionality whilst gaining the ability to change the way the widget is rendered.  The only problem is, I don't yet know how to tell my **ForeignKey** field to use my widget in preference to the default Django widget.  

Note that I don't want to provide a universal override for the **ForeignKey** field, I just want to override it in this one instance.  If I had wanted to override all instances of the template I could simply add the following line to my `views.py` file:

```
django.contrib.admin.widgets.RelatedFieldWidgetWrapper = MyRelatedFieldWidgetWrapper
```

### Find some relevant examples
Searching for **RelatedFieldWidgetWrapper** yielded surprisingly little.  It did, however, give a link to a [set of examples](https://www.programcreek.com/python/example/78782/django.contrib.admin.widgets.RelatedFieldWidgetWrapper) of how others have used the **RelatedFieldWidgetWrapper**.  This turned out to be an extremely useful source of information.  By looking at how other people were using this code I was able to work out what I need to do with it.

### Reproduce the existing functionality
The first step is to reproduce the existing functionality.  I created a new template by copying the [existing template](https://github.com/django/django/blob/8f6860863e34cb1cbe24161f1c4e7c79007e93dc/django/contrib/admin/templates/admin/widgets/related_widget_wrapper.html) into the templates directory for my application.  I called this template `my_related_widget_wrapper.html`.  I then created a bespoke [Class-based view](https://docs.djangoproject.com/en/2.2/topics/class-based-views/) in the `admin.py` file for my application as follows:

```
class MyRelatedFieldWidgetWrapper(RelatedFieldWidgetWrapper):
    template_name = 'my_aplication/my_related_widget_wrapper.html'
    pass
```



### Roll your own

Needed to disable or remove the `showAdminPopup` function in the `Media` associated with the widget.  To do this, removed `?{{ url_params }}` and class `related-widget-wrapper-link` 