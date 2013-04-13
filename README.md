# Banzai

[![Build Status](https://travis-ci.org/patriciomacadden/banzai.png?branch=master)](https://travis-ci.org/patriciomacadden/banzai)
[![Code Climate](https://codeclimate.com/github/patriciomacadden/banzai.png)](https://codeclimate.com/github/patriciomacadden/banzai)

A minimalistic microframework built on top of [Rack](http://rack.github.io/).

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'banzai'
# or this if you want to use master
# gem 'banzai', github: 'patriciomacadden/banzai'
```

And then execute:

```bash
$ bundle
```

Or install it yourself as:

```bash
$ gem install banzai
```

## Features

* DSL inspired by [Sinatra](http://www.sinatrarb.com/).
* Extensible with standard ruby classes and modules, with no extra logic (see
the included modules).

## Usage

`Banzai` applications are just instances of `Banzai::Base`, which complies the
[Rack SPEC](http://rack.rubyforge.org/doc/SPEC.html).

Here is a  classic **Hello World!** example (write this code in `config.ru`):

```ruby
require 'banzai'

class App < Banzai::Base
  get '/' do
    'Hello World!'
  end
end

run App.new
```

### Routes

You can define routes as in [Sinatra](http://www.sinatrarb.com/):

```ruby
class App < Banzai::Base
  get '/' do
    'Hello world'
  end

  get '/hi/:name' do
    "Hello #{request.params[:name]}"
  end
end
```

Every route is composed of a verb, a path and a block. When an incoming request
matches a route, the block is executed and a response is sent back to the
client. The return value of the block will be the `body` of the response. The
`headers` and `status code` of the response will be calculated by
`Rack::Response`, but you could modify it anyway you want it.

Additionally, when a route gets called you have this methods available:

* `env`: The Rack environment.
* `request`: a `Rack::Request` instance.
* `response`: a `Rack::Response` instance.

### Rendering

`Banzai` comes with a module that uses [Tilt](https://github.com/rtomayko/tilt)
for rendering templates. See the example:

In `config.ru`:

```ruby
require 'banzai'

class App < Banzai::Base
  include Banzai::Render

  get '/' do
    render 'views/index.erb'
  end
end

run App.new
```

and in `views/index.erb`:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Hello World!</title>
  </head>
  <body>
    <h1>Hello World!</h1>
  </body>
</html>
```

#### Layout

For now, the `Banzai::Render` module is pretty simple (just `render`). If you
want to render a template within a layout, you could simply do this:

In `config.ru`:

```ruby
require 'banzai'

class App < Banzai::Base
  include Banzai::Render

  get '/' do
    render 'views/layout.erb' do
      render 'views/index.erb'
    end
  end
end

run App.new
```

In `views/layout.erb`:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Hello World!</title>
  </head>
  <body>
    <%= yield %>
  </body>
</html>
```

And in `views/index.erb`:

```html
<h1>Hello World!</h1>
```

#### Partials

Partials are just `render` calls:

```ruby
<%= render 'views/_some_partial.erb' %>
```

#### Helpers

Who needs helpers when you have standard ruby methods? All methods defined in
the application can be used in the templates, since the template code is
executed within the scope of the application instance. See an example:

```ruby
require 'banzai'

class App < Banzai::Base
  include Banzai::Render

  def name
    'World'
  end

  get '/' do
    render 'views/index.erb'
  end
end

run App.new
```

and in `views/index.erb`:

```ruby
<!DOCTYPE html>
<html>
  <head>
    <title>Hello <%= name %>!</title>
  </head>
  <body>
    <h1>Hello <%= name %>!</h1>
  </body>
</html>
```

### Redirecting

If you look at Banzai implementation, you may notice that there is no
`redirect` method (or similar). This is because such functionality is provided
by [Rack::Response](https://github.com/rack/rack/blob/master/lib/rack/response.rb)
and for now we [don't wan't to repeat ourselves](http://en.wikipedia.org/wiki/Don't_repeat_yourself).
So, if you want to redirect to another route, do it like this:

```ruby
require 'banzai'

class App < Banzai::Base
  get '/' do
    response.redirect '/hi'
  end

  get '/hi' do
    'Hello World!'
  end
end

run App.new
```

### Sessions

You can add user sessions using any [Rack session middleware](https://github.com/rack/rack/tree/master/lib/rack/session)
and then access the session through `env['rack.session']`. Fortunately, there
is `Banzai::Session` which comes with a useful helper:

```ruby
require 'banzai'
require 'securerandom'

class App < Banzai::Base
  include Banzai::Session
  use Rack::Session::Cookie, secret: SecureRandom.hex(64)

  get '/' do
    session[:name] = 'banzai'
  end

  get '/' do
    session[:name]
  end
end

run App.new
```

### Rack middleware

Each banzai application is a Rack stack (See this [blog post](http://m.onkey.org/ruby-on-rack-2-the-builder)).
You can add any Rack middleware to the stack by using the `use` class method:

```ruby
require 'banzai'

class App < Banzai::Base
  include Banzai::Session
  use Rack::Session::Cookie, secret: SecureRandom.hex(64)
  use Rack::ShowExceptions

  get '/' do
    session[:name] = 'banzai'
  end

  # more routes...
end

run App
```

### Extending Banzai

You can extend banzai by creating modules or classes. See `Banzai::Render` or
`Banzai::Session` for examples.

## More examples

See the `examples` directory.

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request

## License

See the [LICENSE](https://github.com/patriciomacadden/banzai/blob/master/LICENSE).
