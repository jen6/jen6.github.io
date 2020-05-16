require "rubygems"
require "tmpdir"

require "bundler/setup"
require "jekyll"
#
## Customize post location and post extensions
#

baseurl = `cat ./_config.yml | awk '/baseurl/ { print $2 }' | sed 's/\"//g'`

SOURCE = "."
CONFIG = {
  'posts' => File.join(SOURCE, "_posts"),
  'post_ext' => "md",
  'theme_package_version' => "0.1.0"
}

#
## Color console outout support just because :D
#
module Colors
  def colorize(text, color_code)
    "\033[#{color_code}m#{text}\033[0m"
  end

  {
    :black    => 30,
    :red      => 31,
    :green    => 32,
    :yellow   => 33,
    :blue     => 34,
    :cyan     => 36,
  }.each do |key, color_code|
    define_method key do |text|
      colorize(text, color_code)
    end
  end
end
include Colors

#
## Just typing `rake` will invoke `rake preview`
#
task :default => :preview
load '_rake-configuration.rb' if File.exist?('_rake-configuration.rb')

desc 'Preview with livereload on local machine'
task :preview => :clean do
	puts green "Starting livereload server"
  jekyll('serve -L')
end
task :serve => :preview

desc 'Clean up generated site'
task :clean do
    cleanup
end

desc 'Check links for generated site'
task :check do
  STDOUT.sync = true
	cleanup
  system "bundle exec jekyll build"
	puts cyan "Running html proofer..."
	puts `htmlproofer --assume-extension --alt-ignore '/.*/' ./_site`
end

GITHUB_REPONAME = "jen6/jen6.github.io"
PROD_DESTINATION = "_site"
desc 'generated site'
task :generate do
  STDOUT.sync = true
	cleanup
  system "bundle exec jekyll build"
end

desc 'publishing the site'
task :publish => [:generate] do
    Dir.mktmpdir do |tmp|
    cp_r "#{PROD_DESTINATION}/.", tmp

    pwd = Dir.pwd
    Dir.chdir tmp

    system "git init"
    system "git add ."
    message = "Site updated at #{Time.now.utc}"
    system "git commit -m #{message.inspect}"
    system "git remote add origin https://github.com/#{GITHUB_REPONAME}.git"
    system "git push origin master:refs/heads/master --force"
    system "pause"

    Dir.chdir pwd
  end
end
  

# Usage: rake post title="A Title" [date="2012-02-09"] [tags=[tag1,tag2]] [category="category"]
desc "Begin a new post in #{CONFIG['posts']}"
task :post do
  abort("rake aborted: '#{CONFIG['posts']}' directory not found.") unless FileTest.directory?(CONFIG['posts'])
  title = ENV["title"] || "new-post"
  tags = ENV["tags"] || "[]"
  category = ENV["category"] || ""
  category = "\"#{category.gsub(/-/,' ')}\"" if !category.empty?
  slug = title.downcase.strip.gsub(' ', '-').gsub(/[^\w-]/, '')
  begin
    date = (ENV['date'] ? Time.parse(ENV['date']) : Time.now).strftime('%Y-%m-%d')
  rescue => e
    puts red "Error - date format must be YYYY-MM-DD, please check you typed it correctly!"
    exit -1
  end
  filename = File.join(CONFIG['posts'], "#{date}-#{slug}.#{CONFIG['post_ext']}")
  if File.exist?(filename)
    puts "#{filename} already exists. Do you want to overwrite? : y/n"
    input = STDIN.gets.strip
    if input == "n"
      abort("rake aborted!") 
    end
  end

  puts cyan "Creating new post: #{filename}"
  open(filename, 'w') do |post|
  end

  thumbnail = ""
  File.readlines(filename).each do |li|
    m = li.match /\!\[.*\]\((.*)\)/
    if m
      thumbnail = m[1]
      break
    end
  end

  open(filename, 'w') do |post|
    post.puts "---"
    post.puts "layout: post"
    post.puts "title: \"#{title.gsub(/-/,' ')}\""
    post.puts 'description: ""'
    post.puts "category: #{category}"
    post.puts "tags: #{tags}"
    post.puts "thumbnail: #{thumbnail}"
    post.puts "---"
    post.puts "{% include JB/setup %}"
  end
end # task :post

# Usage: rake post title="A Title" [file="./title.md"] [date="2012-02-09"] [tags=[tag1,tag2]] [category="category"]
desc "Begin a new post in #{CONFIG['posts']}"
task :addinfo do
  abort("rake aborted: '#{CONFIG['posts']}' directory not found.") unless FileTest.directory?(CONFIG['posts'])

  title = ENV["title"]
  filePath = ENV["file"]
  tags = ENV["tags"] || "[]"
  category = ENV["category"] || ""
  category = "\"#{category.gsub(/-/,' ')}\"" if !category.empty?
  slug = title.downcase.strip.gsub(' ', '-').gsub(/[^\w-]/, '')

  begin
    date = (ENV['date'] ? Time.parse(ENV['date']) : Time.now).strftime('%Y-%m-%d')
  rescue => e
    puts red "Error - date format must be YYYY-MM-DD, please check you typed it correctly!"
    exit -1
  end

  filename = File.join(CONFIG['posts'], "#{date}-#{slug}.#{CONFIG['post_ext']}")
  if File.exist?(File.join(CONFIG['posts'], filePath))
    abort("rake aborted! #{filename}") 
  end

  thumbnail = ""
  File.readlines(filePath).each do |li|
    m = li.match /\!\[.*\]\((.*)\)/
    if m
      thumbnail = m[1]
      break
    end
  end

    
  
  puts cyan "Creating new post: #{filename}"
  open(filename, 'w') do |post|
    post.puts "---"
    post.puts "layout: post"
    post.puts "title: \"#{title.gsub(/-/,' ')}\""
    post.puts 'description: ""'
    post.puts "category: #{category}"
    post.puts "tags: #{tags}"
    post.puts "comments: true"
    post.puts "thumbnail: #{thumbnail}"
    post.puts "---"
    File.readlines(filePath).each do |li|
      post.puts li
    end
  end
end # task :post

#
## General support functions
#

def cleanup
  sh 'rm -rf _site'
end

def jekyll(directives = '')
  sh 'jekyll ' + directives
end

def rake_running
  `ps | grep 'rake' | grep -v 'grep' | wc -l`.to_i > 1
end

