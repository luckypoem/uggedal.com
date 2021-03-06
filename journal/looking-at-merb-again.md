% Looking at Merb Again
% 2008-07-27

I'll finally have some spare time to start
[a side project][our] now that I'm delivering my master thesis in a few days.
The question of what web framework to use have been bugging me for some weeks.
I basically think it will come down to whether I want to have fun or if I want
to be as productive as possible. I've narrowed down the "fun" candidates to
[Merb][mer], [Pylons][pyl], and [Werkzeug][wer] (roll my own framework). For
the more productive candidates with loads of plugins and readily available
applications I've looked at [Rails][rai] and [Django][dja].

At this point in time I'm leaning towards Merb and Django. I've used Rails a
fair bit when it was pre 1.0. Frankly, I find it kind of boring as a framework.
Werkzeug may be a bit too low level for my tastes. Pylons looks like a
pythonic Merb to me, but is missing something which makes it stand out from
the crowd.

I'm going to start out with Merb and see how it fares.
I played around with Merb back in the days when it was basically a [Mongrel
handler][mha] with [ERB][erb]. I've kept tab on its progress, but have not
used it after it was split into `merb-core` (bare bones framework) and
`merb-more` (plug in what you need of extra functionality). First off I had to
install the darn thing:

    gem install ParseTree ruby2ruby echoe
    git clone git://github.com/defunkt/sake.git
    cd sake
    rake package
    gem install pkg/sake-1.0.16.gem
    cd ..
    sake -i "http://merbivore.com/merb-dev.sake"
    sake merb:clone
    cd merb
    export MERB_SUDO=""
    gem install rspec json_pure erubis mime-types rack mongrel hpricot
    sake merb:install:extlib
    sake merb:install:core
    sake merb:install:more

I used [sake][sak] with a [Rake][rak] script which allowed me to nicely clone
all the needed Merb repos and then build and install packages from them. To
keeping up to date with `HEAD` I would then only have to run
`sake merb:update && sake merb:install`. Unfortunately the latest sake release
had some dependency problems and I had to install it from `HEAD`. On a
side note I hate it when libs like `echoe` are constructed to only work with
`sudo`. Hence the manual `gem install` step.

What are the immediate good parts of Merb? One nice feature seems to be the
fact that controllers are plain old Ruby objects. This
makes for easily testable controllers:

    class Exercises < Merb::Controller
      def index
        @exercises = Exercise.all(:order => [:title.asc])
        display @exercies
      end
    end

    describe Exercises, '#index' do
      it 'should provide a sorted list of exercises' do
        Exercise.should_receive(:all).with(:order => [:title.asc])

        @exercises = mock(:exercises)

        dispatch_to(Exercises, :index) do |controller|
          controller.should_receive(:display).with(@exercises)
        end
      end
    end

Because of the Merb controller design the `dispatch_to` method is simply
implemented as:

    def dispatch_to(controller, action, params = {}, env = {}, &blk)
      dispatch_request(build_request(params, env), controller, action, &blk)
    end

    def dispatch_request(request, controller, action, &blk)
      this_controller = controller.new(request)
      yield this_controller if block_given?
      this_controller._dispatch(action)

      this_controller
    end

These methods were edited slightly for brevity. `build_request` simply builds
a fake request with the given parameters. As you can see controller are simply
instantiated with this fake request and dispatched to the right action before
its returned. A simple and transparent design which means that one can easily
interact with the controller in separation from views and models.

The routing engine of Merb is also very powerful. It makes the simple tasks easy
and more complex tasks possible:

    Merb::Router.prepare do |r|
      # REST resource with:
      #   GET: index, show, new, edit, delete
      #   POST: create
      #   PUT: update
      #   DELETE: destroy
      r.resources :exercises

      # Regex matches with request environment checks:
      r.match(%r{/openid/(.+)}, :user_agent => /iPhone/)
          .to(:controller => 'session', :action => '[1]')
    end

Diving into `merb-more` we find the optional `merb-actions-args` which makes
it possible to define controller methods with arguments. This eliminates the
need for the `params` hash and makes for cleaner code:

    class Exercises < Merb::Controller

      def update(id, article)
        @exercise = Excercise[id]

        if @exercise.update_attribtues(article)
          redirect url(:exercise, @exercise)
        else
          render :edit
        end
      end
    end

Another feature that feels just simple and right is the way you can provide
alternative formats from your controllers:

    class Exercises < Merb::Controller
      provides :json

      def show(id)
        @exercise = Excercise[id]
        display @exercise
      end
    end

The `provides` API works by either looking at the HTTP Accepts header of the
incoming request or an URI with a format suffix. The following request
should thus return the same results:

    curl -H "Accept: application/json" http://ourlifts.com/exercises/23
    curl http://ourlifts.com/exercises/23.json

How does Merb transform the `@exerciese` object into different formats? It
first looks for a view (template) matching the action (method) with a
format suffix like `show.html.haml` or `show.json.erb`. If no template
for the matching format is found Merb simply calls `to_json` or `to_html`
on the object destined for display. Most often one would have a
template for HTML format and let Merb call `to_json` on the object
for API or `XMLHttpRequest` calls.

Merb recently added a clean way to send messages to the next rendered page
like Rails' [flash][fla]. Unlike Rails' flash Merb's messages are sent with
the controllers `redirect` method:

    class Exercises < Merb::Controller

      def update(id, article)
        @exercise = Excercise[id]

        if @exercise.update_attribtues(article)
          redirect url(:exercise, @exercise), "#{@exercise.title} updated!"
        else
          render :edit
        end
      end
    end

Merb marshals and base64 encodes the message and sends it as a
`_message` query parameter unlike Rails' reliance on a throwaway session.
Merb also makes the message available through the `@message` variable in your
views.

The last feature of Merb that I'll highlight here is the recent *Slices*
plugin. [Merb Slices][mes] are separate applications which can be mounted into
a base Merb application. The slice can then be customized by for example
modifying its templates to fit better with your specific application.

This reminds me of [reusable Django apps][rda] where one can use one or many
of the loads of freely available Django apps to rapidly build a new
application. Unfortunately there seem to be very few Merb slices available.
The only really useful slice I could locate was a
[user authentication slice][uas].

[mer]: http://merbivore.com
[pyl]: http://pylonshq.com
[wer]: http://werkzeug.pocoo.org
[rai]: http://rails.org
[dja]: http://djangoproject.com
[mha]: http://mongrel.rubyforge.org/web/mongrel/classes/Mongrel/HttpHandler.html
[erb]: http://www.ruby-doc.org/stdlib/libdoc/erb/rdoc/
[our]: http://ourlifts.com
[sak]: http://github.com/defunkt/sake
[rak]: http://rake.rubyforge.org
[mes]: http://github.com/wycats/merb-more/tree/master/merb-slices
[rda]: http://www.b-list.org/weblog/2007/mar/27/reusable-django-apps/
[uas]: http://github.com/hassox/merb-auth/tree/master
[fla]: http://api.rubyonrails.org/classes/ActionController/Flash.html
