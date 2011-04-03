---
layout: post
title: Classy Web Development with Sinatra and Heroku
excerpt: Since my recent switch from WordPress to Jekyll, I've really enjoyed playing around with Ruby. Although I don't have much programming experience, I've been fascinated with one particular Ruby framework, Sinatra.
comments: true
---
<img src="/images/sinatra.gif" class="alignleft">Since my recent switch from WordPress to Jekyll, I've really enjoyed playing around with Ruby. Although I don't have much programming experience, I've been fascinated with one particular Ruby framework, <a href="http://www.sinatrarb.com/" rel="external" target="_new">Sinatra</a>.
	
Sinatra is a DSL (domain-specific language) for quickly creating web applications in Ruby with very little effort. It is a minimalist framework, built right on top of <a href="http://rack.rubyforge.org/" rel="external" target="_new">Rack</a>, a standard interface for Ruby web frameworks. Unlike rails, you won't find many bells and whistles here. There are no models, views or controllers by default, but that's where Sinatra really shines. You can use Sinatra to create lean, focused web applications in just a few lines of code.

So this morning, I published a Sinatra [template](https://github.com/bummercloud/sinatra-heroku-template) for Heroku with Haml, Sass & jQuery. The repository includes a base HTML 5 template for Sinatra, ready for Heroku deployment. Use `bundle install` to grab all of the gem dependencies. If you get lost during deployment, check out [Getting Started with Heroku](http://devcenter.heroku.com/articles/quickstart)

One of the fun parts of the project was using Haml in place of erb. I'm still pulling up the Haml reference here and there but it's a breeze to work with. Here is an example of the layout page:

{% highlight haml %}
!!! 5
%html{html_attrs('en-en')}
  %head
    %meta{:'http-equiv' => "Content-Type", :content => "text/html; charset=utf-8"}
    %meta{:name => "lang", :content => "en"}
    %title Sinatra Template for Heroku w/ Haml, Sass & jQuery | TJ Stein
    %link{:href => "assets/css/style.css", :rel => "stylesheet", :type => "text/css"}/
  %body
    .container
    = yield
    = haml :analytics, :layout => false
{% endhighlight %}

The actual application file doesn't have much in it. It loads the required gems, builds the routes including the Sass support. You can add or modify any of the routes to create new pages or functions:

{% highlight ruby %}
require 'sinatra'
require 'haml'
require 'sass'

set :haml, :format => :html5

get '/' do
  haml :index
end

get '/:path' do
  haml params[:path].to_sym
end

get '/style.css' do
  content_type 'text/css', :charset => 'utf-8'
  scss :style
end
{% endhighlight %}

If you prefer using a Rakefile to deploy, I've included an example Rakefile with a rake task for deployment to Heroku using `rake deploy`:

{% highlight ruby %}
require 'rake'

desc "Deploy to Heroku."
task :deploy do
   require 'heroku'
   require 'heroku/command'
   user, pass = File.read(File.expand_path("~/.heroku/credentials")).split("\n")
   heroku = Heroku::Client.new(user, pass)

   cmd = Heroku::Command::BaseWithApp.new([])
   remotes = cmd.git_remotes(File.dirname(__FILE__) + "/../..")

   remote, app = remotes.detect {|key, value| value == (ENV['APP'] || cmd.app)}

   if remote.nil?
   raise "Could not find a git remote for the '#{ENV['APP']}' app"
   end

   `git push #{remote} master`

   heroku.restart(app)
end
{% endhighlight %}

If you end up using the template in your own application or want to contribute, send a pull request or just fork the project. Keep it classy.