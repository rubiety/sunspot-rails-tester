# sunspot-rails-tester

This gem allows you to "turn on" solr for certain portions
of your tests. For the code that does not use solr, you
would want to "stub" sunspot to avoid unneeded indexing.

Here is an example RSpec 2 spec_helper.rb:

    $original_sunspot_session = Sunspot.session

    RSpec.configure do |config|
      config.mock_with :rspec
      
      config.before do
        Sunspot.session = Sunspot::Rails::StubSessionProxy.new($original_sunspot_session)
      end
      
      config.before :solr => true do
        Sunspot::Rails::Tester.fork_original_sunspot_session
        Sunspot.session = $original_sunspot_session
        Sunspot.remove_all!
      end
    end
    
Let's go through what the above code does.

* `$original_sunspot_session` stores the _original_ sunspot
  session. By default, sunspot_rails uses the `SessionProxy::ThreadLocalSessionProxy`.
  
* In the first `before` block, we set the session to a stub session for
  _every_ example. `Sunspot::Rails::StubSessionProxy` is just a dummy class
  that skips indexing.
  
* In the second `before` block, we use RSpec 2's metadata feature by
  adding `:solr => true`. Any example or example group with this metadata
  will run the _original_ sunspot session.
  `Sunspot::Rails::Tester.fork_original_sunspot_session` starts the solr instance
  if it's not running.
  
Here is an example spec that utilizes sunspot-rails-tester:

    require 'spec_helper'
    
    describe 'search page' do
      it 'highlights the active tab in the navigation' do
        # uses the stub session
      end
      
      it 'finds and displays a person', :solr => true do
        # uses actual solr - indexing will happen
      end
    end
    
## Thanks

The following articles served as guidance and inspiration for this gem:

* [http://blog.kabisa.nl/2010/02/03/running-cucumber-features-with-sunspot_rails/](http://blog.kabisa.nl/2010/02/03/running-cucumber-features-with-sunspot_rails/)
* [http://opensoul.org/blog/archives/2010/04/07/cucumber-and-sunspot/](http://opensoul.org/blog/archives/2010/04/07/cucumber-and-sunspot/)
    
## Copyright

Copyright (c) 2011 Justin Ko. See LICENSE for details.