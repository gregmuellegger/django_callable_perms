About
=====

django_callable_perms implements a modular registry of permission handlers.
Handlers can be registered as simple permission callbacks, following
the Django has_perm parameters.

All permission callbacks may implement the permission checks they need. As
we talk about callbacks, meaning executable code, you can put everything
you need in there.

Example
=======

Django itself misses support for row level permissions. You may add this using
authentication backends and there are existing apps to implement this. Anyways
most of these solutions will add some overhead and cannot provide really
flexible permissions.

django_callable_perms do not care about which permissions you want to implement
but fit row level permissions really well. So lets start with the real example.
Given a model like this:

::
    
    class Article(models.Model):
        title = models.CharField(max_length=100)
        author = models.ForeignKey(User)

You may want to add permissions checks, so only the author is allowed to
edit his/her articles. Now django_callable_perms may be used to add simple
checks like the following:

::
    
    # may be put into permissions.py inside the app
    from django_callable_perms import register
    from .models import Article
    
    
    def author_may_always_edit(user, perm, obj):
        # no additional database query needed!
        if obj.author_id == user.pk:
            return True
    
    
    register(
        'app.change_article', # permission name
        author_may_always_edit, # callback function
        Article, # model, for which the permission check is implemented
                 # may be "None", if no instance is required
    )

Permission checks afterwards just follow default Django behavior. Use
{% get_obj_perms %} inside templates.

::
    
    {% get_obj_perms for user accessing obj as obj_perms %}
    {% if obj_perms.app.change_article %}
        {# show edit link #}
        <a href="...">Edit</a>
    {% endif %}

Note: If you need your own permissions you do not need to add them to the
database (see Django docs), most of the time it's enough to just register
the new permissions.

Installation
============

Add django_callable_perms to INSTALLED_APPS (for autoloading) and add
django_callable_perms.backend.CallablePermissionBackend to your
AUTHENTICATION_BACKENDS.

Make sure to put django_callable_perms into your Python PATH first, of
course.

Additional Features
===================

Autoloading
+++++++++++

Will load all permissions.py's inside the INSTALLED_APPS.

sync_permissions
++++++++++++++++

Management command to create all app permission inside the database. May be
used to initialize the database. Will not (meaning never) add more than the
default Django permissions.

