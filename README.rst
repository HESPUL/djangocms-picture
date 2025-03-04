============================================
Manual rollback of responsive modifications.
============================================

Not so much used, so remove modifications and restore back ``djangocms-picture`` as dependency.

Run this SQL to drop columns and remove migrations.



    DROP INDEX IF EXISTS djangocms_picture_picture_large_screen_picture_id_ec59f427;
    DROP INDEX IF EXISTS djangocms_picture_picture_medium_screen_picture_id_32e1e218;
    
    ALTER TABLE djangocms_picture_picture DROP COLUMN alternative_format_webp;
    ALTER TABLE djangocms_picture_picture DROP COLUMN large_screen_picture_id;
    ALTER TABLE djangocms_picture_picture DROP COLUMN large_screen_viewport_width;
    ALTER TABLE djangocms_picture_picture DROP COLUMN medium_screen_picture_id;
    ALTER TABLE djangocms_picture_picture DROP COLUMN medium_screen_viewport_width;
    ALTER TABLE djangocms_picture_picture DROP COLUMN small_screen_viewport_width;
    
    DELETE FROM django_migrations WHERE app='djangocms_picture' AND name='0013_merge_20240502_1754';
    DELETE FROM django_migrations WHERE app='djangocms_picture' AND name='0012_auto_20220504_2117';




=============================
django CMS Picture Responsive
=============================

|pypi| |build| |coverage|

**django CMS Picture Responsive** is a fork of then **django CMS Picture** plugin for `django CMS <http://django-cms.org>`_
that allows you to add images on your site.

This fork adds options to allow CMS users to define 3 different images according to the browser screen size.

.. note::

        The original plugin is endorsed by the `django CMS Association <https://www.django-cms.org/en/about-us/>`_.

.. note::
        
        I will try to maitain this package and follow upstream evolutions

.. image:: preview.gif


Documentation
=============

See ``REQUIREMENTS`` in the `setup.py <https://github.com/divio/djangocms-picture/blob/master/setup.py>`_
file for additional dependencies:

|python| |django| |djangocms|

* Django Filer 1.7 or higher

Make sure `django-filer <http://django-filer.readthedocs.io/en/latest/installation.html>`_
is installed and configured appropriately.


Installation
------------

For a manual install:

* run ``pip install djangocms-picture``
* add ``djangocms_picture`` to your ``INSTALLED_APPS``
* run ``python manage.py migrate djangocms_picture``


Configuration
-------------

Note that the provided templates are very minimal by design. You are encouraged
to adapt and override them to your project's requirements.

This addon provides a ``default`` template for all instances. You can provide
additional template choices by adding a ``DJANGOCMS_PICTURE_TEMPLATES``
setting::

    DJANGOCMS_PICTURE_TEMPLATES = [
        ('background', _('Background image')),
    ]

You'll need to create the `background` folder inside ``templates/djangocms_picture/``
otherwise you will get a *template does not exist* error. You can do this by
copying the ``default`` folder inside that directory and renaming it to
``background``.

Another setting is ``DJANGOCMS_PICTURE_NESTING``, which allows you to render an image
as the background image of a container that also contains other content (text, icons
and so on). ::

    DJANGOCMS_PICTURE_NESTING = True

will enable this (the default is ``False``). When set to ``True``, you'll be able to place additional
plugins inside the picture plugin.

You can override alignment styles with ``DJANGOCMS_PICTURE_ALIGN``, for example::

    DJANGOCMS_PICTURE_ALIGN = [
        ('top', _('Top Aligned')),
    ]

This will generate a class prefixed with ``align-``. The example above
would produce a ``class="align-top"``. Adding a ``class`` key to the image
attributes automatically merges the alignment with the attribute class.

When using the ``DJANGOCMS_PICTURE_ALIGN`` setting, you have the flexibility to align images in various styles, such as left, right, or center, as well as to float images and vertically align them. You can customize these alignment options as follows::

    DJANGOCMS_PICTURE_ALIGN = [        
        ('left', _('Align left')),
        ('right', _('Align right')),
        ('center', _('Align center')),
        # Image float alignment options
        ("start", _("Float left")),            
        ("end", _("Float right")),
        # Vertical alignment options
        ('top', _('Align top')),
        ('middle', _('Align middle')),
        ('bottom', _('Align Bottom')),
        ('baseline', _('Align baseline')),           
    ]

This configuration will generate a CSS class prefixed with ``align-``. For example, when selecting the left align option, it will produce a class attribute like ``class="align-left"`` for the image. You can apply these classes to your image attributes, and they will automatically handle the alignment.

These alignment options are consistent with `Bootstrap's image alignment styles <https://getbootstrap.com/docs/5.3/content/images/#aligning-images>`_, however, **you have to provide your own CSS styles for the alignment options**, be it through Bootstrap or otherwise.

You can enable responsive images technique by setting``DJANGOCMS_PICTURE_RESPONSIVE_IMAGES_ENABLED`` to ``True``.
In this case uploaded images will create thumbnails of different sizes according
to ``DJANGOCMS_PICTURE_RESPONSIVE_IMAGES_SIZES`` (which defaults to ``[576, 768, 992]``) and browser
will be responsible for choosing the best image to display (based upon the screen viewport).

Therefore, the plugin accept 2 alternatives pictures for medium and large screen (small being the default picture). 
If you do, the plugin will generate a ``<picture>`` HTML tag including the different sources with display conditions
based on viewport ``min-width`` breakpoints. The settings ``DJANGOCMS_PICTURE_RESPONSIVE_IMAGES_BREAKPOINT_MEDIUM`` and ``DJANGOCMS_PICTURE_RESPONSIVE_IMAGES_BREAKPOINT_LARGE``
(in px, which defaults to 768 and 992) can be used to specify the size of the medium and large viewport breakpoints. 
You can also specify in the plugin the expected viewport width percent part (vw) the image will cover.
This will increase browser ability to choose the right image (choosing a smaller version instead of full width one).

Using responsive mode also allow to automatically generate a WebP version of the images. 
There is a checkbox to specify independently for each image, but the default value  
can be set using the ``DJANGOCMS_PICTURE_RESPONSIVE_IMAGES_ALTERNATIVE_FORMAT_WEBP`` setting.
(which default to ``False``)
`Pillow require <https://pillow.readthedocs.io/en/stable/installation.html#external-libraries>` the libwebp library to be installed on the host system.

`See MDN documentation for more informations on responsive images <https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images>`_

You can use ``DJANGOCMS_PICTURE_RATIO`` to set the width/height ratio of images
if these values are not set explicitly on the image::

    DJANGOCMS_PICTURE_RATIO = 1.618

We use the `golden ratio <https://en.wikipedia.org/wiki/golden_ratio>`_,
approximately 1.618, as a default value for this.

When working out sizes for the image, the system will use the following values,
of preference:

* the width or height set in the *Thumbnail options*
* *Autoscale*
* the *Width* and *Height*

We recommend setting width or height values around a placeholder so
when the plugin uses *Autoscale* it can discover them::

    {% with 720 as width and 480 as height %}
        {% placeholder content %}
    {% endwith %}

Further configuration can be achieved through the
`django Filer settings <https://django-filer.readthedocs.io/en/latest/settings.html>`_.


Running Tests
-------------

You can run tests by executing::

    virtualenv env
    source env/bin/activate
    pip install -r test_requirements/base.txt
    python setup.py test


.. |pypi| image:: https://badge.fury.io/py/djangocms-picture-responsive.svg
    :target: http://badge.fury.io/py/djangocms-picture-responsive
.. |build| image:: https://travis-ci.org/divio/djangocms-picture.svg?branch=master
    :target: https://travis-ci.org/divio/djangocms-picture
.. |coverage| image:: https://codecov.io/gh/divio/djangocms-picture/branch/master/graph/badge.svg
    :target: https://codecov.io/gh/divio/djangocms-picture

.. |python| image:: https://img.shields.io/badge/python-3.5+-blue.svg
    :target: https://pypi.org/project/djangocms-picture/
.. |django| image:: https://img.shields.io/badge/django-2.2,%203.0,%203.1-blue.svg
    :target: https://www.djangoproject.com/
.. |djangocms| image:: https://img.shields.io/badge/django%20CMS-3.7%2B-blue.svg
    :target: https://www.django-cms.org/


Updating from `cmsplugin-filer <https://github.com/django-cms/cmsplugin-filer>`_
--------------------------------------------------------------------------------

Historically, `cmsplugin-filer` was used to create file, folder, image, link, teaser & video plugins on your django CMS projects. Now `cmsplugin-filer` has been archived, you can still migrate your old instances without having to copy them manually to the new `djangocms-<file|picture|link|...>` plugins.

There's a third-party management command that supports your migration:

`migrate_cmsplugin_filer.py <https://gist.github.com/corentinbettiol/84a6ea7e4d047fc01861b0af15fd60f0>`_

This management command is only a starting point. It *has* worked out of the box for some people, but we encourage you to read the code, understand what it does, and test it on a development environment before running it on your production server.

The management command is only configured to transfer your `cmsplugin_link`, `cmsplugin_file`, `cmsplugin_folder` and `cmsplugin_image` plugins to modern `djangocms_*` plugins. If you need to transfer other `cmsplugin_*` plugins, you'll have to write your own code.

Alternatively you can use the `deprecate_cmsplugin_filer <https://github.com/ImaginaryLandscape/deprecate_cmsplugin_filer>`_ app, which only adds a small migration that transfer the old `cmsplugin-filer` plugins instances to the new `djangocms-<file|picture|link|...>` plugins.
