% Efficient and Lightweight Notices in Django
% 2008-10-20

Drawing inspiration from [Merb][mer] I wrote a lightweight messaging app for
Django. My solution uses a normal GET query parameter
containing the base64 and urlencoded message.
[django-notices][not] supports sending messages to the next page when
a HTTP redirect is used. The developer API thus looks like this:

    from notices import HttpResponseRedirectWithNotice
    from django.shortcuts import render_to_response

    def add_counter(req, number):
        if number > 0:
            Counter.objects.create(number=number)
            notice = "Created counter with %s" % number
            return HttpResponseRedirectWithNotice("/", success=notice)
        else:
            return render_to_response('form.html', {
                'error_message': "Counters have to be positive"
            })

To handle these messages in a template one could do something like this:


    {% if notices %}
      <ul class="notices">
        {% for type, notice in notices.items %}
          <li class="{{ type }}">{{ notice }}</li>
        {% endfor %}
      </ul>
    {% endif %}

By default HttpResponseRedirectWithNotice supports notices with the type
of success, error, and notice. This can be changed in settings.py:

    NOTICE_TYPES = ("success", "failure")

An alternative to django-notices are the [bundled messages system][aut]
in django.contrib.auth. My main grief with this solution is
that it only supports one type of message to be sent. Another
problem is that is uses the database, which results in one extra
query for each page view.

There are cookie based solutions inspired by Rails available for
Django, but I don't like using cookies to handle such state.

[mer]: http://github.com/wycats/merb-core/tree/d4003c983169052551eb884f456600fc826bfa55/lib/merb-core/controller/mixins/controller.rb#L117
[not]: http://github.com/uggedal/django-notices
[aut]: http://docs.djangoproject.com/en/dev/topics/auth/#messages
