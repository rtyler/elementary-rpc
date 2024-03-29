# Elementary RPC

Elementary RPC is a basic
[Protobuf](https://developers.google.com/protocol-buffers/docs/overview) HTTP
RPC gem which aims to provide:

 * Easy RPC usage
 * Parallelism by default
 * An easily extended RPC request pipeline

### Sample Usage

For the following Protobuf RPC service definition:

```
package echoserv;

message String {
  required string data  = 1;
  optional int64 status = 2;
}

service Simple {
  rpc Echo (String) returns (String);
  rpc Reverse (String) returns (String);
}
```

A corresponding Ruby client might look something like this:

```ruby
require 'rubygems'
require 'elementary'

require 'echoserv/service.pb' # Include our protobuf object declarations


hosts = [{:host => 'localhost', :port => '9292', :prefix => '/rpcserv'}]


# Let's use the Statsd middleware to send RPC timing and count information to 
# Graphite (this presumes we have already used `Statsd.create_instance` elsewhere
# in our code)
Elementary.use(Elementary::Middleware::Statsd, :client => Statsd.instance)


# Create our Connection object that knows about our Protobuf service
# definition
c = Elementary::Connection.new(Echoserv::Simple, :hosts => hosts)

# Create a Protobuf message to send over RPC
msg = Echoserv::String.new(:data => str)


echoed = c.rpc.echo(msg) # => Elementary::Future
reversed = c.rpc.reverse(msg) # => Elementary::Future

sleep 10 # Twiddle our thumbs doing other things

puts {
  :echoed => echoed.value, # resolve the future and get our value out
  :reversed => reversed.value,
}.to_json
```

## Installation

Add this line to your application's Gemfile:

    gem 'elementary-rpc'

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install elementary-rpc

## Usage

TODO: Write usage instructions here

## Contributing

1. Fork it ( https://github.com/[my-github-username]/elementary-rpc/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request
