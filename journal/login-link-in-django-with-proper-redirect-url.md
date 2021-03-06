%  Login Link in Django with Proper Redirect URL
% 2009-04-15

When using Django's `contrib.auth` I tend to use the `@login_required`
decorator for restricting access to sensitive views. Whats nice about
its implementation is that it will redirect me to a login page and remember
the URL I tried to access. After I've
successfully logged in it will redirect me to this URL.

Like many people I tend to place a login link at the top of every page if
the current user is not authenticated and a log out link otherwise:

    {% if user.is_authenticated %}
      <a href="{% url auth_logout %}">Log out</a>
    {% else %}
      <a href="{% url auth_login %}">Log in</a>
    {% endif %}

The problem with this naive approach is that if I'm on a page with the
URL `http://example.org/super/duper/` and I click "Log in", I'm taken to
whatever the setting `LOGIN_REDIRECT_URL` dictates (`/accounts/profile/`
by default) after a successful authentication process. Caring for the
usability of my site I naturally wanted a successfull login attempt to
take the user back to the page where he or she clicked my "Log in" link.

A neat solution for this problem is to use a [context processor][con] to
populate each [RequestContext][req] with a login URL with the proper
redirect parameter set. Note that this requires you to use `RequestContext`
instead of the plain `Context` in your views when you're rendering
templates (I use generic views or a [shortcut for render_to_response][sho]
to handle this).

Implementing such a context processor can easily be achieved by adding the
following code to a file called `context_processors.py` inside one of your
Django applications:

    from django.utils.http import urlquote
    from django.conf import settings
    from django.contrib.auth import REDIRECT_FIELD_NAME

    def login_url_with_redirect(request):
        login_url = settings.LOGIN_URL
        path = urlquote(request.get_full_path())
        url = '%s?%s=%s' % (settings.LOGIN_URL, REDIRECT_FIELD_NAME, path)
        return {'login_url': url}

Then you should add this new context processor to the list of template
context processors in your settings file:

    TEMPLATE_CONTEXT_PROCESSORS = (
        "django.core.context_processors.auth",
        "django.core.context_processors.debug",
        "django.core.context_processors.i18n",
        "django.core.context_processors.media",

        "yourapp.context_processors.login_url_with_redirect',
    )

All that is left to do is to use the new login URL in a template. This could
look something like this:

    {% if user.is_authenticated %}
      <a href="{% url auth_logout %}">Log out</a>
    {% else %}
      <a href="{{ login_url }}">Log in</a>
    {% endif %}

[req]: http://docs.djangoproject.com/en/dev/ref/templates/api/#subclassing-context-requestcontext
[con]: http://docs.djangoproject.com/en/dev/ref/settings/#setting-TEMPLATE_CONTEXT_PROCESSORS
[sho]: http://www.djangosnippets.org/snippets/3/
