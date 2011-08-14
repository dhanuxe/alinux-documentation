
require 'rubygems'
require 'rake'
require 'jekyll'
require 'fileutils'
require 'yaml'
require 'time' 

ROOT_DIR = File.dirname(__FILE__)
SITE_DIR = File.join(ROOT_DIR, '_site')
TAGS_DIR = File.join(ROOT_DIR, 'tags')
DRAFTS_DIR = File.join(ROOT_DIR, '_drafts')
POSTS_DIR = File.join(ROOT_DIR, '_posts')

PUBLISH_HOST = "dhanuxe@store.alinux.web.id"
PUBLISH_PATH = "~/www/alinux/"

options = Jekyll.configuration({})
site = Jekyll::Site.new(options)
site.read_posts('')

## Clean garbage vim
desc "Cleaning Garbage vim..."
task :vimclean do
	puts "Cleaning Garbage vim..."
	sh "find `pwd` -name \"*~\" -exec rm -rf {} \\\;"
end

## syncing to plox
desc "Syncing to plox"
task :rsync do
	#sh "cd _site;rsync -avu --relative -e \"ssh -C\" '.' dhanuxe@store.alinux.web.id:~/www/alinux/"
 	 sh "rsync -avz --delete #{SITE_DIR}/ #{PUBLISH_HOST}:#{PUBLISH_PATH}"
end

# Generate tag cloud page
# -----------------------
task :cloud do
    puts 'Generating tag cloud...'

    html =<<-HTML
---
layout: cloud
title: Tag cloud
---

<p>Click on a tag to see the relevant posts.</p>
HTML

    html << '<nav id="post-archive-category-list">'
    site.categories.sort.each do |category, posts|
      font_size = 12 + (posts.count*1.5);
      html << "<a href=\"javascript:filterByCategory('#{category}')\" class=\"#{category} \" style=\"font-size: #{font_size}px\">#{category}</a>\n"
    end
    html << '<a class="reset selected" href="javascript:filterByCategory(\'reset\')">ALL</a></nav>
    <br/>
      <nav id="post-archive-list">
          <ul>
	  	{% for post in site.posts  %}
		<li class="{% for tag in post.categories %} {{ tag }}{% endfor %}"><a href="{{ site.pet }}{{ post.url }}">{{ post.title | xml_escape }}</a> - <span>{{ post.date | date: "%b %d %Y" }}</span></li>
				{% endfor %}
				    </ul>
      </nav>
    
    
    '

    File.open('tags/cloud.html', 'w+') do |file|
      file.puts html
    end

    puts 'Done. File written to: ./tags/cloud.html'
end

# Generate tags page
# ------------------
task :tags do
    puts "Generating tags..."

    site.categories.sort.each do |category, posts|
        html =<<-HTML
---
layout: default_new
title: Entries tagged "#{category}"
---

        <section class="nice-link-list">
            <header>
                <h1>{{ page.title }}</h1>
            </header>

		<br/>
            <ul>
                {% for post in site.categories['#{category}'] %}
                <li><a href="{{ site.pet }}{{ post.url }}">{{ post.title }}</a> - <time datetime="{{ post.date | date: "%Y-%m-%d" }}" pubdate>{{ post.date | date_to_string }}</time></li>
                {% endfor %}
            </ul>
		<br/>
            <p>Go to the <a href="{{ site.pet }}/tags/cloud.html">tag cloud</a> to see other tags..</p>
HTML

        File.open("tags/#{category}.html", 'w+') do |file|
          file.puts html
        end
    end
    puts 'Done. Files written to: ./tags/'
end

# Generate website
# ----------------
#task :generate => [:cloud, :tags] do
#    sh 'jekyll'
#end

# Clean dir www
#task :clean do
#	sh 'rm -rf /var/www/techno/*'
#end



desc "Clear generated site."
task :clean do
  rm_rf Dir.glob(File.join(SITE_DIR, '*'))
  rm_rf Dir.glob(File.join(TAGS_DIR, '*'))
end

# Generate website
# ----------------
task :taggen => [:cloud, :tags] do
    puts 'Generated tag cloud and all tag pages.'
    sh "touch tags/index.html"
end

desc "Generate site."
task :build do
  sh "jekyll"
end

desc "Run local jekyll server"
task :server, [:port] do |t, args|
  sh "jekyll --server #{args.port || 4000} --base-url /alinux --auto"
end

desc "Publish site."
task :publish => [:vimclean, :clean, :cloud, :tags, :build] do |t|
 # sh "rsync -avz --delete #{SITE_DIR}/ #{PUBLISH_HOST}:#{PUBLISH_PATH}"
# sh "rm -rf /usr/local/nginx/html/*;cp -a _site/* /usr/local/nginx/html/"
 sh "rm -rf /var/www/htdocs/alinux/*;cp -a _site/* /var/www/htdocs/alinux/"
 # puts "Commit your posts and changes.\nThen run:\n  git push origin master"
end

def parse_post(post)
  raise Exception.new("Invalid post file format") unless post[0] = "---"
  eoh = post[1, post.length].index("---\n") + 1
  header = YAML.load(post[1, eoh].to_s)
  body = post[eoh + 1, post.length]
  return header, body
end

def write_post(header, body, file)
  File.open(file, "w") do |f|
    f << YAML::dump(header)
    f << "---\n"
    f << body
  end
end

def edit_post(post)
   editor = ENV['VISUAL'] || ENV['EDITOR']
  # editor = ENV['EDITOR']
   system "#{editor} #{post}"  
end

desc "Create a new draft post"
task :draft, [:title] do |t, args|
  title = args.title || "Untitled"

  # Create a new file with a basic template
  postname = title.strip.downcase.gsub(/ /, '-')
  post = File.join(DRAFTS_DIR, "#{postname}.markdown")

  header = <<-END
---
title: #{title}
layout: post_new
categories: [Uncategories]
---

New draft post

END

  mkdir_p(DRAFTS_DIR)
  File.open(post, 'w') {|f| f << header }  
  edit_post(post)

  puts "Created draft post #{post}."
  puts "To publish, use:"
  puts "  rake post[#{postname}]"
end

desc "Publish draft post. Arguments: [title]"
task :post, [:title] do |t, args|
  unless args.title
    puts "Usage: rake post[\"Title\"]"
    exit(-1)
  end

  published_timestamp = Time.now
  #names = header["title"].strip.downcase.gsub(/ /, '-')
  date_prefix = published_timestamp.strftime("%Y-%m-%d")
  draft_path = File.join(DRAFTS_DIR, "#{args.title}.markdown")
  header, body = parse_post(IO.readlines(draft_path))

  # Create the post file
  name = header["title"].strip.downcase.gsub(/ /, '-')
  header["date"] = published_timestamp.strftime("%Y-%m-%d %H:%M:%S") unless header.include?("date")
  post_path = File.join(POSTS_DIR, "#{date_prefix}-#{name}.markdown")
  write_post(header, body, post_path)
  
  # Clear draft
  File.delete(draft_path)
 
  puts "Posted: #{post_path}"
end

desc "Update published post. Arguments: [title]"
task :update, [:title] do |t, args|
  unless args.title
    puts "Usage: rake post[\"Title\"]"
    exit(-1)
  end

  updated_timestamp = Time.now
  postname = args.title.strip.downcase.gsub(/ /, '-')

 # name = header["title"].strip.downcase.gsub(/ /, '-')
 # header["date"] = published_timestamp.strftime("%Y-%m-%d %H:%M:%S") unless header.include?("date")

  published_timestamp = Time.now
  date_prefix = published_timestamp.strftime("%Y-%m-%d")

  post_path = File.join(POSTS_DIR, "#{date_prefix}-#{postname}.markdown")
  header, body = parse_post(IO.readlines(post_path))
  header["updated"] = updated_timestamp.strftime("%Y-%m-%d %H:%M:%S")
  write_post(header, body, post_path)
  edit_post(post_path)
  puts "Updated: #{post_path}"
end
