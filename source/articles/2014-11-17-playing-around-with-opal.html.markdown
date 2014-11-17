---
title: Playing around with Opal
date: 2014-11-17 20:55 UTC
tags: Ruby, Javascript
---

Alright, so before I even get started here, I've got to make some
disclosures. Not 'somebody is paying me for these words' disclosures
mind you, but more 'please don't comment rage on my words'. 

So I want to make clear, I'm a fan of Javascript. Mostly, all of the cool
things you can do in a web browser with it. Paired with CSS3, the
animations you can pull off are downright impressive these days, and
that's awesome.

There is just one problem: I don't really like writing Javascript.

See, I spend most of my time writing Ruby, and I love it. Javascript
still has all these braces and semicolons. Then, wait, my function isn't
in the right place. I'll need to hoist that. Grumble.

Ok, yes, I can, and should, be using Coffeescript. Yes, that's better,
much better. It feels more like ruby, pushes a little harder on the OOP
that I know and love. However it's still not Ruby.

So what is a programmer to do? Enter <a href='http://opalrb.org' target='_blank'>Opal</a>

What is Opal? Well, in a nutshell it's pretty simple. In the same way
that Coffeescript compiles into Javascript, Opal compiles Ruby into
Javascript. Sound cool? It is. Read on.

When I want to play around with a new web technology, I generally build
out a <a href='http://www.sinatrarb.com/' target="_blank">Sinatra</a> project, 
so thats what I'll use in this case. Opal plays well with Rails as well, 
if you'd rather go that route. Or, you can just use it by itself. Go
nuts.

So, what all do we need to get started here? Let's start with the
Gemfile.


  ```ruby
    // Gemfile

    source 'https://rubygems.org'
    gem 'sinatra'
    gem 'opal'
    gem 'opal-jquery'
  ```


Strictly speaking, you don't need opal-jquery to get started, but if
you're doing much work on the front-end, you'll probably want it. Also,
I'll show you a bit of it's functionality here. 

Now, lets set up our Sinatra app. There are any number of ways you might
do this, so understand I'm only demonstrating one approach. Create a
config.ru file to hold you application initialization info.


  ```ruby
    // config.ru

    require 'sinatra'
    require 'opal'
    require 'opal-jquery'

    get '/' do
      haml :index
    end

    opal = Opal::Server.new {|s|
      s.append_path 'app'
      s.main = 'application'
    }

    map opal.source_maps.prefix do
      run opal.source_maps
    end

    map '/assets' do
      run opal.sprockets
    end

    run Sinatra::Application
  ```


What's happening here? After we require all our necessary dependencies,
we define a single route, and then fire up Opal, which will look in our yet to be created app
directory, and will compile the file 'application.rb' into Javascript.
Opal maps this compiled JS to '/assets/application.js', which we'll have
to link to in our html. Go ahead and fire up the application. I'm going
to do so with shotgun, which will reload our changes to the application
without requiring us to restart the server. Alternatively, you could use
`rackup config.ru`


  ```bash
    // from the command line

    gem install shotgun
    bundle install
    shotgun config.ru
  ```


In your browser, navigate to localhost:9393. You should be seeing an error, 
because we haven't created our index view. Lets do that now. Be sure to 
create a directory for the views as well.


  ```haml
    // /views/index.haml

    !!! 5
    %html
      %head
        %script(src='/assets/application.js')
        %link{rel: :stylesheet, type: :'text/css', href: url('/styles.css') }
      %body
  ```
    

Reload the page, and you should see a blank screen. Two things here:
we're now loading in the JS that Opal compiled, as well as a yet to be
created stylesheet. Go ahead and create an empty stylesheet under a new
directory `/public/styles.css`

Generally, I would suggest using scss or sass rather than plain css, but
we're not going to be doing much here, so we'll keep it simple.

Ok, now that sinatra is running properly for us, and we have a view to
play on, lets write some ruby! Create your application file under a new
directory `/app/application.rb`. If you remember in the `config.ru`
file, this is what Opal is looking for to compile into JS. Add the
following:


  ```ruby
    // /app/application.rb

    require 'opal'
    require 'jquery'
    require 'opal-jquery'

    Document.ready? do
      
      alert 'This alert was written in Ruby'

    end
  ```

Reload the page, and voila! Nothing happens.

So, enter one of the minor annoyances I've run into thus far using Opal. In
order for everything to work properly, Opal needs an actual copy of
jQuery in the app directory, the jQuery gem won't do the trick. So this
is easy enough to solve. Download a copy of <a
href='http://jquery.com' target='_blank'>jQuery</a> and save it in your
app directory as `/app/jquery.js`. Do this and reload the page.

Hey! Now that's pretty sweet eh? A Javascript alert that we didn't write
a JS to make.

Now, alerts are all good and well, but JS and Ruby have far more functionality
than that, so lets dig in a bit. Make the following chages to your
project:

  ```haml
    // /views/index.haml

    !!! 5
    %html
      %head
        %script(src='/assets/application.js')
        %link{rel: :stylesheet, type: :'text/css', href: url('/styles.css') }
      %body
        #color_box
  ```


  ```css
    // /public/styles.css

    #color_box {
      width: 200px;
      height: 200px;
      position: absolute;
      top: 50%;
      left: 50%;
      background-color: red;
    }
  ```


  ```ruby
    // /app/application.rb

    require 'opal'
    require 'jquery'
    require 'opal-jquery'

    Document.ready? do
      
      box = Element['#color_box']

      box.on(:click) do
        height, width = get_random_sizes
        box.animate(height: height, width: width, speed: 500)
      end
      
    end

    def get_random_sizes
      [rand(50..400), rand(50..400)]
    end
  ```

After making your changes, you should see a red square on the center of
the blank page. Try clicking on it, and it should animate to a different
size. Pretty snazzy huh?

Taking a closer look at our `application.rb` file, there are a few
things to take note of. We're using `Document.ready?` the same way you
would if you were writing JS, and the class `Element` represents DOM
elements. We use the Element class convenience method `[]`, to find
elements, but we could also `Element.find()`

We also make use of the jQuery methods 'on' and 'animate', but get to
use a friendly Ruby block sytax. Also, you'll see we defined our own
helper function to get some random sizes. We could just as easily start
to bring in our own custom objects and classes here.

Now this is all cool, and fairly contrived, but one big piece is
missing. When I use javascript, I frequently find myself needing AJAX
functionality. Opal makes this stupid simple.

Add the following route to config.ru:

  ```ruby
    // config.ru

    get '/colors' do
      { colors: ['green', 'purple', 'blue', 'yellow'] }.to_json
    end
  ```

And adjust the `Document.ready?` block in application.rb:

  ```ruby
    // application.rb

    Document.ready? do
      box = Element['#color_box']
    
      box.on(:click) do
        set_color(box)
      end
    end
    
    def set_color(element)
      HTTP.get('/colors') do |response|
        color = JSON.parse(response.body)['colors'].sample
        element.css('background-color', color)
      end
    end
  ```

Opal provides us with this HTTP class to give access to all the AJAXy
goodness such as get, post, put, and delete. In this example, we're
really just fetching a list of colors and changing the background color
on our element, but you can imagine doing much more complex tasks.

If you're interested in learning more about this great project, I highly
recommend the <a href='http://opalrb.org/docs/'>Opal Documentation</a>.
Hopefully you've found this useful, and as always, feel free to reach out if
you have any questions.

If you'd like to take a look at any of the code
here, I've got it up on <a
href='https://github.com/wvmitchell/opal_experiment'>Github</a> (although it may have evolved slightly by
the time you get there).

- Will
