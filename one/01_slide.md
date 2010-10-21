!SLIDE
# Crafting A Somewhat Useful Command Line Gem

!SLIDE bullets incremental
# Synopsis

* APIs should utilize 304 Not Modified responses
* Testing with `curl` is hard


!SLIDE commandline small
# Synopsis

    $ curl -I --digest -u test@example.com:pass \
        -H "Accept: application/json" \
        "http://my.cloudapp.local/items?page=1&per_page=5"


    HTTP/1.1 401 Authorization Required
    Status:           401
    Content-Type:     application/json; charset=utf-8
    WWW-Authenticate: Digest realm="Application", qop="auth", algorithm=MD5,
                      nonce="blahblahblah", opaque="blahblahblah"

    HTTP/1.1 200 OK
    Status:        200
    Content-Type:  application/json; charset=utf-8
    ETag:          "c6b5d4a0589c53bf1d8326422fc33f57"
    Last-Modified: Fri, 15 Oct 2010 14:46:14 GMT


!SLIDE commandline small
# Synopsis

    $ curl -I --digest -u test@example.com:pass \
        -H "Accept: application/json" \
        -H "If-None-Match: \"c6b5d4a0589c53bf1d8326422fc33f57\"" \
        -H "If-Modified-Since: Fri, 15 Oct 2010 14:46:14 GMT" \
        "http://my.cloudapp.local/items?page=1&per_page=5"


    HTTP/1.1 401 Authorization Required
    Status:           401
    Content-Type:     application/json; charset=utf-8
    WWW-Authenticate: Digest realm="Application", qop="auth", algorithm=MD5,
                      nonce="blahblahblah", opaque="blahblahblah"

    HTTP/1.1 304 Not Modified
    ETag: "c6b5d4a0589c53bf1d8326422fc33f57"


!SLIDE commandline
# Solution

    $ smog auth=test@example.com:pass \
        "http://my.cloudapp.local/items?page=1&per_page=5"


    curl -I -s --digest -u test@example.com:pass \
        -H "Accept: application/json" \
        "http://my.cloudapp.local/items?page=1&per_page=5"

        HTTP/1.1 401 Authorization Required
        HTTP/1.1 200 OK

    curl -I -s --digest -u test@example.com:pass \
        -H "Accept: application/json" \
        -H "If-None-Match: \"c6b5d4a0589c53bf1d8326422fc33f57\"" \
        -H "If-Modified-Since: Fri, 15 Oct 2010 14:46:14 GMT" \
        "http://my.cloudapp.local/items?page=1&per_page=5"

        HTTP/1.1 401 Authorization Required
        HTTP/1.1 304 Not Modified


!SLIDE commandline
# Initialize Gem

    $ bundle gem smog
          create  smog/Gemfile
          create  smog/Rakefile
          create  smog/.gitignore
          create  smog/smog.gemspec
          create  smog/lib/smog.rb
          create  smog/lib/smog/version.rb
    Initializating git repo in /Users/Larry/smog


!SLIDE code smaller
# `smog.gemspec`

    @@@ ruby
    Gem::Specification.new do |s|
      # snip

      # Installed with: `gem install smog`
      s.add_dependency "main"

      # Installed with: `gem install smog --development`
      s.add_development_dependency "rspec"
      s.add_development_dependency "cucumber"
      s.add_development_dependency "aruba"

      # snip
    end


!SLIDE commandline
# Solution

    $ smog auth=test@example.com:pass \
        "http://my.cloudapp.local/items?page=1&per_page=5"


    curl -I -s --digest -u test@example.com:pass \
        -H "Accept: application/json" \
        "http://my.cloudapp.local/items?page=1&per_page=5"

        HTTP/1.1 401 Authorization Required
        HTTP/1.1 200 OK

    curl -I -s --digest -u test@example.com:pass \
        -H "Accept: application/json" \
        -H "If-None-Match: \"c6b5d4a0589c53bf1d8326422fc33f57\"" \
        -H "If-Modified-Since: Fri, 15 Oct 2010 14:46:14 GMT" \
        "http://my.cloudapp.local/items?page=1&per_page=5"

        HTTP/1.1 401 Authorization Required
        HTTP/1.1 304 Not Modified


!SLIDE code smaller
# `features/smog.feature`

    @@@ ruby
    Scenario: Get items
      When I run the command:
        """
        smog auth=test@example.com:pass \
            "http://my.cloudapp.local/items?page=1&per_page=5"
        """
      Then the output should be exactly:
        """
        curl -I -s --digest -u test@example.com:pass \
            -H "Accept: application/json" \
            "http://my.cloudapp.local/items?page=1&per_page=5" \
            HTTP/1.1 401 Authorization Required
            HTTP/1.1 200 OK

        curl -I -s --digest -u test@example.com:pass 
            -H "Accept: application/json" \
            -H "If-None-Match: \"c6b5d4a0589c53bf1d8326422fc33f57\"" 
            -H "If-Modified-Since: Fri, 15 Oct 2010 14:46:14 GMT"
            "http://my.cloudapp.local/items?page=1&per_page=5" 
            HTTP/1.1 401 Authorization Required
            HTTP/1.1 304 Not Modified
        """

!SLIDE code smaller
# `features/smog.feature`

    @@@ ruby
    Then the output should contain:
      """
      curl -I -s --digest -u test@example.com:pass \
          -H "Accept: application/json" \
          "http://my.cloudapp.local/items?page=1&per_page=5" \
          HTTP/1.1 401 Authorization Required
          HTTP/1.1 200 OK
      """
    And the output should match:
      """
      curl -I -s --digest -u test@example.com:pass \
          -H "Accept: application/json" \
          -H "If-None-Match: \\"[^"]+\\"" \
          -H "If-Modified-Since: [\w,: ]+" \
          "http://my.cloudapp.local/items\?page=1&per_page=5"
          HTTP/1.1 401 Authorization Required
          HTTP/1.1 304 Not Modified
      """


!SLIDE code small
# `lib/smog/cli.rb`

    @@@ ruby
    require 'main'

    Main do
      version Smog::VERSION

      argument 'url' do
        description 'url to query'
      end

      keyword 'auth' do
        required
        description 'digest auth credentials'
      end

      def run
        # Insert magic here
      end
    end


!SLIDE commandline
# `lib/smog/cli.rb`

    $ bundle exec smog --help

    NAME
      smog v0.0.2

    SYNOPSIS
      smog url auth=auth [options]+

    PARAMETERS
      url (1 -> url) 
          url to query 
      auth=auth (1 -> auth) 
          digest auth credentials 
      --help, -h


!SLIDE code smaller
# `lib/smog/cli.rb`

    @@@ ruby
    def build_curl
      [ 'curl -I -s',
        "--digest -u #{ params[:auth].value }",
        header('Accept', 'application/json'),
        params[:url].value.inspect
      ].join ' '
    end

    def header(name, value)
      %{-H "#{ name }: #{ value }"}
    end

    def run
      command = build_curl

      @last_response = %x(#{ command })
      puts_response command, @last_response
    end


!SLIDE commandline
# `lib/smog/cli.rb`

    $ bundle exec smog auth=test@example.com:pass \
        "http://my.cloudapp.local/items?page=1&per_page=5"


    curl -I -s --digest -u test@example.com:pass \
        -H "Accept: application/json" \
        "http://my.cloudapp.local/items?page=1&per_page=5"

        HTTP/1.1 401 Authorization Required
        HTTP/1.1 200 OK


!SLIDE code smaller
# `lib/smog/cli.rb`

    @@@ ruby
    def run
      2.times do
        command = build_curl

        @last_response = %x(#{ command })
        puts_response command, @last_response
      end
    end

    def build_curl
      [ 'curl -I -s',
        "--digest -u #{ params[:auth].value }",
        header('Accept', 'application/json')
      ].tap do |command|
        if etag
          command << header('If-None-Match', %{\\"#{ etag }\\"})
        end

        if last_modified
          command << header('If-Modified-Since', last_modified)
        end

        command << params[:url].value.inspect
      end.join ' '
    end


!SLIDE code smaller
# `lib/smog/cli.rb`

    @@@ ruby
    def etag
      response_header /ETag: "(.*)"/
    end

    def last_modified
      response_header /Last-Modified: (.*)/
    end

    def response_header(match_header)
      return unless @last_response

      @last_response.match(match_header)[1].chomp
    end


!SLIDE commandline
# `lib/smog/cli.rb`

    $ bundle exec smog auth=test@example.com:pass \
        "http://my.cloudapp.local/items?page=1&per_page=5"


    curl -I -s --digest -u test@example.com:pass \
        -H "Accept: application/json" \
        "http://my.cloudapp.local/items?page=1&per_page=5"

        HTTP/1.1 401 Authorization Required
        HTTP/1.1 200 OK

    curl -I -s --digest -u test@example.com:pass \
        -H "Accept: application/json" \
        -H "If-None-Match: \"c6b5d4a0589c53bf1d8326422fc33f57\"" \
        -H "If-Modified-Since: Fri, 15 Oct 2010 14:46:14 GMT" \
        "http://my.cloudapp.local/items?page=1&per_page=5"

        HTTP/1.1 401 Authorization Required
        HTTP/1.1 304 Not Modified


!SLIDE commandline
# Release

    $ rake release

    smog 0.0.4 built to pkg/smog-0.0.4.gem
    Tagged aa835647 with v0.0.4
    Pushed git commits and tags
    Pushed smog 0.0.4 to rubygems.org


!SLIDE commandline
# Fo' Rizzle

    $ gem install smog
    Successfully installed smog-0.0.4
    1 gem installed


    $ smog auth=test@example.com:pass "http://my.cloudapp.local/items?page=1&per_page=5"
    curl -I -s --digest -u test@example.com:pass -H "Accept: application/json" "http://my.cloudapp.local/items?page=1&per_page=5"
        HTTP/1.1 401 Authorization Required
        HTTP/1.1 200 OK

    curl -I -s --digest -u test@example.com:pass -H "Accept: application/json" -H "If-None-Match: \"c6b5d4a0589c53bf1d8326422fc33f57\"" -H "If-Modified-Since: Fri, 15 Oct 2010 14:46:14 GMT" "http://my.cloudapp.local/items?page=1&per_page=5"
        HTTP/1.1 401 Authorization Required
        HTTP/1.1 304 Not Modified


!SLIDE bullets
# Links

* Bundler: [http://github.com/carlhuda/bundler](http://github.com/carlhuda/bundler)
* Aruba: [http://github.com/aslakhellesoy/aruba](http://github.com/aslakhellesoy/aruba)
* Main: [http://github.com/ahoward/main](http://github.com/ahoward/main)
* Smog: [http://github.com/lmarburger/smog](http://github.com/lmarburger/smog)
