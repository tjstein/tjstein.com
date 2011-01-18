# adapted from 
# https://github.com/mmonteleone/michaelmonteleone.net/blob/master/Rakefile
# and
# https://github.com/jbarratt/serialized.net/blob/master/Rakefile

require 'rake/clean'

desc 'Build site with Jekyll'
task :build => [:clean] do
  # compile site
  jekyll  
end

desc 'Notify Google of the new sitemap'
task :sitemap do
  begin
    require 'net/http'
    require 'uri'
    puts '* Pinging Google about the sitemap'
    Net::HTTP.get('www.google.com', '/webmasters/tools/ping?sitemap=' + URI.escape('http://tjstein.com/sitemap.xml'))
  rescue LoadError
    puts '! Could not ping Google about our sitemap, because Net::HTTP or URI could not be found.'
  end
end
 
desc 'Start server with --auto'
task :server => [:clean]  do
  jekyll('--server --auto')
end

desc 'Build and deploy'
task :deploy => :build do
  sh 'rsync -rtz --delete _site/ deploy@tjstein.com:/var/www/tjstein.com/public'
end

desc 'Push source code to Github'
task :push do
  puts '* Pushing to Github'
  puts `git push origin master`
end

desc 'List all draft posts'
task :drafts do
  puts `find ./_posts -type f -exec grep -H 'published: false' {} \\;`
end

desc 'Begin a new post'
task :post do   
  ROOT_DIR = File.dirname(__FILE__)

  title = ARGV[1]
  tags = ARGV[2 ]

  unless title
    puts %{Usage: rake post "The Post Title"}
    exit(-1)
  end

  datetime = Time.now.strftime('%Y-%m-%d')  # 30 minutes from now.
  slug = title.strip.downcase.gsub(/ /, '-')

  # E.g. 2006-07-16_11-41-batch-open-urls-from-clipboard.markdown
  path = "#{ROOT_DIR}/_posts/#{datetime}-#{slug}.markdown"

  header = <<-END
---
layout: post
title: #{title}
excerpt: Description Content
comments: true
---

END

  File.open(path, 'w') {|f| f << header }
  system("mate", "-a", path)    
end  

task :default => :server

def jekyll(opts = '')
  sh 'jekyll ' + opts
end