.. _apphooks_introduction:

########
Apphooks
########

Right now, our Django Polls application is statically hooked into the project's
``urls.py``. This is all right, but we can do more, by attaching applications to
django CMS pages.


*****************
Create an apphook
*****************

We do this with an **apphook**, created using a :class:`CMSApp
<cms.app_base.CMSApp>` sub-class, which tells the CMS how to include that application.

Apphooks live in a file called ``cms_apps.py``, so create one in your Polls/CMS Integration
application, i.e. in ``polls_cms_integration``.

This is a very basic example of an apphook for a django CMS application:

.. code-block:: python

    from cms.app_base import CMSApp
    from cms.apphook_pool import apphook_pool
    from django.utils.translation import ugettext_lazy as _


    @apphook_pool.register  # register the application
    class PollsApphook(CMSApp):
        app_name = "polls"
        name = _("Polls Application")

        def get_urls(self, page=None, language=None, **kwargs):
            return ["polls.urls"]


In this ``PollsApphook`` class, we have done several key things:

* ``app_name`` attribute gives the system a unique way to refer to the apphook - see
  :ref:`multi_apphook` for details on why this matters.
* ``name`` is a human-readable name, and will be displayed to the admin user.
* ``get_urls()`` method is what actually hooks the application in, returning a
  list of URL configurations that will be made active wherever the apphook is used.

**Restart the runserver**. This is necessary because we have created a new file containing Python
code that won't be loaded until the server restarts. You only have to do this the first time the
new file has been created.


.. _apply_apphook:

***************************
Apply the apphook to a page
***************************

Now we need to create a new page, and attach the Polls application to it through this apphook.

Create and save a new page, then publish it.

..  note:: Your apphook won't work until the page has been published.

In its *Advanced settings*, choose "Polls Application" from the *Application* menu, and save once
more.

.. image:: /introduction/images/select-application.png
   :alt: select the 'Polls' application
   :width: 400
   :align: center

Refresh the page, and you'll find that the Polls application is now available
directly from the new django CMS page.

You can now remove the mention of the Polls application (``url(r'^polls/', include('polls.urls',
namespace='polls'))``) from your project's ``urls.py`` - it's no longer even required there,
because we reach the polls via the apphook instead. If you leave it there, then you'll receive a
warning in the logs::

    URL namespace 'polls' isn't unique. You may not be able to reverse all URLs in this namespace.

..  important::

    Don't add child pages to a page with an apphook.

    The apphook "swallows" all URLs below that of the page, handing them over to the attached
    application. If you have any child pages of the apphooked page, django CMS will not be
    able to serve them reliably.
