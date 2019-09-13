---
title: "Overriding Django Widget Templates"
meta: "A simple method for overriding widget templates in Django. Specifically, this describes how to override the RelatedFieldWidgetWrapper for ForeignKey and ManyToMany fields."
categories: Django
slug: "overriding-django-widget-templates"
---

# Overriding Django Widget Templates
I spent all day yesterday trying to find a quick and easy way to override the widget wrapper for a ForeignKey field in a Django Admin form.  Despite too many internet searches to mention I couldn't find a quick, easy and comprehensible way of achieving my goal.  I eventually found a way to override the widget template by combining everything I'd read and sleeping on it.  This post is the result.

## Aim
I have a model which links to itself via a parent-child relationship.  I want to add properties to this relationship so I have created an [intermediate model]() to maintain links between parents and children using [ForeignKeys]().  When editing a parent, I display the associated children using a [TabularInline]() admin model.

The default widget for ForeignKey fields within a TabularInline is the [RelatedFieldWidgetWrapper]().  This widget is almost what I need, but not quite.  The issue for me is that the default behaviour for creating new children or editing existing children is to open a popup.  Since the model being created/edited is identical to the parent model, I could easily find myself in a very confusing loop whereby I create/edit a child in a popup and then create/edit another child for that child in another popup... etcetera, ad infinitum.

I want to to avoid this by changing the URLs associated with the add/edit actions to replace the change page for the parent model currently being edited with a change page for the new/updated child model.  I'll be adding a breadcrumb at the top of these change pages so that I can easily navigate back to the parent and will be auto-saving parents and children at appropriate points in the process.  The details of the breadcrumbs and model lifecycle are not the topic of this post.  Overriding the links in the default widget *is*.

## Method
All I need to do is to override the default widget template html.  The interwebs appears not to contain any *comprehensible* information on how to do this.  [StackOverflow]() contains lots of answers to this particular question, but these are mostly obscure, over-complicated or simply lacking in sufficient detail for those of us who are not already Django experts.  Here is my humble contribution.

### Examine the Default Implementation

### Find some relevant examples

### Reproduce the existing functionality

### Roll your own

Needed to disable or remove the `showAdminPopup` function in the `Media` associated with the widget.  To do this, removed `?{{ url_params }}` and class `related-widget-wrapper-link` 