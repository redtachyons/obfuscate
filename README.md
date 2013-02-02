# Obfuscate

A simple way to obfuscate ids and text. Useful when you have to make ids visible
to users. Integrates directly with Rails 3.

The goal is to make simple obfuscated ids that are not huge. This is achieved by using
Blowfish and encrypting a single block. This produces a nice id of 11 characters (11
since the trailing = is removed by default), for example `3NINgAbOhPc`. The caveat is,
the id must be with 99,999,999, e.g. a max length of 8.

Text can be obfuscated using Blowfishes string encryption as well, but than it produces
output that is larger than the elegant 11 character single block encryption.

https://github.com/mguymon/obfuscate

[RDoc](http://rubydoc.info/gems/obfuscate/frames)

## Install

    gem install obfuscate

## Ruby Usage

A simple example

    Obfuscate.setup do |config|
      config.salt = "A weak salt ..."
      config.mode = :string # defaults to :string
    end

    obfuscated = Obfuscate.obfuscate( "test" )
    clarified = Obfuscate.clarify( obfuscated )

## Rails Integration

Create an initializer in `config/initializers` with:

    require 'obfuscate/obfuscatable'
    Obfuscate.setup do |config|
      config.salt = "A weak salt ..."
    end

Now add to models that you want to be Obfuscatable:

    class Message < ActiveRecord::Base
      obfuscatable # a hash of config overrides can be passed.
    end

To get the 11 character obfuscated id, which uses `mode :block` for the Blowfish single block encryption:

    message = Message.find(1)
    obfuscated = message.obfuscated_id
    clarified = message.clarify_id( obfuscated )

Or obfuscate a block of text, using Blowfish string encryption, allowing longer than 8 to be obfuscated,
but generate larger obfuscated text:

    obfuscated = message.obfuscate( "if you use your imagination, this is a long block of text" )
    clarified = message.clarify( obfuscated )

## Benchmarks

Some benchmarks from my dev machine, with cpu clocking in at 2.53ghz:

    Obfuscate.setup :salt => 'a very weak salt indead.'

    puts Benchmark.bm { |bm|
      ids = []
      bm.report("obfuscate x1000:") { 
        1000.times { ids << Obfuscate.obfuscate( rand(1...99999999), {:mode => :block} ) }
      }
    
      bm.report("clarify x1000:") {
        ids.each { |id| Obfuscate.clarify( id, {:mode => :block} ) } 
      }
    }
    
    user     system      total        real
    obfuscate x1000: 27.110000   0.000000  27.110000 ( 27.145539)
    clarify x1000: 27.270000   0.000000  27.270000 ( 27.304084)


## License

Licensed to the Apache Software Foundation (ASF) under one or more
contributor license agreements.  See the NOTICE file distributed with this
work for additional information regarding copyright ownership.  The ASF
licenses this file to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance with the License.
You may obtain a copy of the License at

  http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.  See the
License for the specific language governing permissions and limitations under
the License.
