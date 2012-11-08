#!/usr/bin/env ruby

require 'rake'
require 'ostruct'
require 'yaml'
require 'hpricot'
require 'tilt'
require 'haml'
require 'fileutils'
require 'pry'

task :default => :render

task :render do
  config = parse_config

  # Copy reveal.js
  Dir.mktmpdir do |dest|
    FileUtils.rm_r(dest) if Dir.exists?(dest)
    FileUtils.cp_r 'reveal.js', dest

    # Load the templates
    @doc = load_template
    # Modify the values
    update_doc("//meta[@name=author]",config.author,"content")
    update_doc("//meta[@name=description]",config.description.strip,"content")
    update_doc("//title",config.title)

    puts "Rendering templates"
    section_template = Tilt.new('templates/section.html.haml')
    slides = []
    Dir.glob("slides/**").sort.each do |slide|
      content = Tilt.new(slide).render(self)
      slides << section_template.render(self, :content => content)
    end

    update_doc("div.slides","\n#{slides.join("\n")}\n")

    File.open(File.join(dest,'index.html'),'w') do |f|
      f.write @doc.to_html
    end

    # Publishing the docs to gh-pages
    if ENV['publish']
      gh_pages(dest)
    else
      FileUtils.rm_rf 'out'
      FileUtils.cp_r dest, 'out'
    end
  end
end

# TODO: This sometimes leaves artifacts, there should be a better way
def gh_pages(dir)
  stashed = `git stash`
  `git checkout --orphan gh-pages && git rm -rf .`
  `echo "reveal.js\n.gitignore" > .gitignore`
  `cp -R #{dir}/* .`
  `git add .`
  `git commit -m "First commit"`
  `git push origin gh-pages -f`
ensure
  `git checkout master`
  `git branch -D gh-pages`
  unless stashed.match(/No local changes/)
    `git stash pop`
  end
end

def parse_config
  puts "Parsing configuration"
  OpenStruct.new(YAML.load_file('config.yml'))
end

def load_template
  puts "Loading reveal.js template"
  open('reveal.js/index.html'){|f| Hpricot(f)}
end

def update_doc(tag,value,attr = nil)
  el = @doc.at(tag)
  attr.nil? ?
    el.innerHTML = value :
    el.attributes[attr] = value
end
