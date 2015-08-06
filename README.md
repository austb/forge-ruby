#Puppet Forge

Access and manipulate the [Puppet Forge API](https://forgeapi.puppetlabs.com)
from Ruby.

##Installation

Add this line to your application's Gemfile:

    gem 'puppet_forge'

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install puppet_forge

##Requirements

* Ruby >= 1.9.3

##Dependencies

* [Faraday]() ~> 0.9.0
* [Typhoeus](https://github.com/typhoeus/typhoeus) ~> 0.7.0 (optional)

Typhoeus will be used as the HTTP adapter if it is available, otherwise
Net::HTTP will be used. We recommend using Typhoeus for production-level
applications.

##Usage

First, make sure you have imported the Puppet Forge gem into your application:

``` ruby
require 'puppet_forge'
```

Next, supply a user-agent string to identify requests sent by your application
to the Puppet Forge API:

``` ruby
PuppetForge.user_agent = "MyApp/1.0.0"
```

Now you can make use of the resource models defined by the gem:

* [PuppetForge::V3::User][user_ref]
* [PuppetForge::V3::Module][module_ref]
* [PuppetForge::V3::Release][release_ref]

For convenience, these classes are also aliased as:

* [PuppetForge::User][user_ref]
* [PuppetForge::Module][module_ref]
* [PuppetForge::Release][release_ref]

[user_ref]: https://github.com/puppetlabs/forge-ruby/wiki/Resource-Reference#puppetforgeuser
[module_ref]: https://github.com/puppetlabs/forge-ruby/wiki/Resource-Reference#puppetforgemodule
[release_ref]: https://github.com/puppetlabs/forge-ruby/wiki/Resource-Reference#puppetforgerelease

__The aliases will always point to the most modern API implementations for each
model.__ You may also use the fully qualified class names
(e.g. PuppetForge::V3::User) to ensure your code is forward compatible.

See the [Basic Interface](#basic-interface) section below for how to perform
common tasks with these models.

Please note that PuppetForge models are identified by unique slugs rather
than numerical identifiers.

The slug format, properties, associations, and methods available on each
resource model are documented on the [Resource Reference][resource_ref] page.

[resource_ref]: https://github.com/puppetlabs/forge-ruby/wiki/Resource-Reference

###Basic Interface

Each of the models uses ActiveRecord-like REST functionality to map over the Forge API endpoints.
Most simple ActiveRecord-style interactions function as intended.

Currently, only unauthenticated read-only actions are supported.

``` ruby
# Find a Resource by Slug
PuppetForge::User.find('puppetlabs') # => #<Forge::V3::User(/v3/users/puppetlabs)>

# Find All Resources
PuppetForge::Module.all # See "Paginated Collections" below for important info about enumerating resource sets.

# Find Resources with Conditions
PuppetForge::Module.where(query: 'apache') # See "Paginated Collections" below for important info about enumerating resource sets.
PuppetForge::Module.where(query: 'apache').first # => #<Forge::V3::Module(/v3/modules/puppetlabs-apache)>
```

###Paginated Collections

The Forge API only returns paginated collections as of v3.

``` ruby
PuppetForge::Module.all.total # => 1728
PuppetForge::Module.all.length # => 20
```

Movement through the collection can be simulated using the `limit` and `offset`
parameters, but it's generally preferable to leverage the pagination links
provided by the API. For convenience, pagination links are exposed by the
library.

``` ruby
PuppetForge::Module.all.offset # => 0
PuppetForge::Module.all.next_url # => "/v3/modules?limit=20&offset=20"
PuppetForge::Module.all.next.offset # => 20
```

An enumerator exists for iterating over the entire (unpaginated) collection.
Keep in mind that this will result in multiple calls to the Forge API.

``` ruby
PuppetForge::Module.where(query: 'php').total # => 37
PuppetForge::Module.where(query: 'php').unpaginated # => #<Enumerator>
PuppetForge::Module.where(query: 'php').unpaginated.to_a.length # => 37
```

###Associations & Lazy Attributes

Associated models are accessible in the API by property name.

``` ruby
PuppetForge::Module.find('puppetlabs-apache').owner # => #<Forge::V3::User(/v3/users/puppetlabs)>
```

Properties of associated models are then loaded lazily.

``` ruby
user = PuppetForge::Module.find('puppetlabs-apache').owner

# This does not trigger a request
user.username # => "puppetlabs"

# This *does* trigger a request
user.created_at # => "2010-05-19 05:46:26 -0700"
```

##Caveats

This library currently does no response caching of its own, instead opting to
re-issue every request each time. This will be changed in a later release.

##Reporting Issues

Please report problems, issues, and feature requests on the public
[Puppet Labs issue tracker][issues] under the FORGE project. You will need
to create a free account to add new tickets.

[issues]: https://tickets.puppetlabs.com/browse/FORGE

##Contributing

1. Fork it ( https://github.com/[my-github-username]/forge-ruby/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request

##Contributors

* Pieter van de Bruggen, Puppet Labs
* Jesse Scott, Puppet Labs
