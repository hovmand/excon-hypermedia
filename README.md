# Excon::Hypermedia [![wercker status](https://app.wercker.com/status/f3fd6cf2045566072ef26354d5a73e9f/s/master "wercker status")](https://app.wercker.com/project/bykey/f3fd6cf2045566072ef26354d5a73e9f)

Teaches [Excon][] how to talk to [HyperMedia APIs][hypermedia].

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'excon-hypermedia'
```

And then execute:

```shell
bundle
```

Or install it yourself as:

```shell
gem install excon-hypermedia
```

## Usage

To let Excon know the API supports HyperMedia, simply enable the correct
middleware (either globally, or per-connection):

```ruby
Excon.defaults[:middlewares].push(Excon::HyperMedia::Middleware)

api = Excon.get('https://www.example.org/api.json')
api.class # => Excon::Response
```

Using the `HyperMedia` middleware, the `Excon::Response` object now knows how
to handle the HyperMedia aspect of the API:

```ruby
product = api.product(expand: { uid: 'bicycle' })
product.class # => Excon::Connection

response = product.get
response.class # => Excon::Response
response.body.class # => String
```

As seen above, you can expand URI Template variables using the `expand` option,
provided by the [`excon-addressable` library][excon-addressable].

Since each new resource is simply an `Excon::Response` object, accessed through
the default `Excon::Connection` object, all [Excon-provided options][options]
are available as well:

```ruby
product.get(idempotent: true, retry_limit: 6)
```

### Links

You can access all links in a resource using the `links` method:

```ruby
api.links.first.class # => Excon::HyperMedia::Link
api.links.first.name  # => 'product'
api.links.first.href  # => 'https://www.example.org/product/{uid}'
```

You can also access a link directly, using its name:

```ruby
api.link('product').href # => 'https://www.example.org/product/{uid}'
```

### Attributes

Attributes are available through the `attributes` method:

```ruby
product.attributes.to_h # => { uid: 'bicycle', stock: 5 }
product.attributes.uid  # => 'bicycle'
```

Attributes can be accessed directly on the `Excon::Response` object, but keep
in mind that this might conflict with existing methods on the response object,
resulting in unexpected return values, so use this sparsely:

```ruby
product.class # => Excon::Response

# resource attribute:
product.stock # => 5

# not an attribute, but the `Excon::Response#status` value:
product.status # => 200

# better separation of concern:
product.attributes.stock # => 5
product.status # => 200
```

## License

The gem is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).

## TODO

* properly handle curied-links and/or non-valid Ruby method name links

[excon]: https://github.com/excon/excon
[hypermedia]: https://en.wikipedia.org/wiki/HATEOAS
[excon-addressable]: https://github.com/JeanMertz/excon-addressable
[options]: https://github.com/excon/excon#options
