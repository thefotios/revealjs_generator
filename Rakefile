#!/usr/bin/env ruby

require 'rake'
require 'ostruct'
require 'yaml'
require 'hpricot'
require 'tilt'
require 'haml'
require 'fileutils'

task :default => :render

task :render do
  config = parse_config

  # Copy reveal.js
  dest = ENV['dir'] || "out"
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
    slides << section_template.render(self, :content => File.read(slide))
  end

  update_doc("div.slides","\n#{slides.join("\n")}\n")

  File.open(File.join(dest,'index.html'),'w') do |f|
    f.write @doc.to_html
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
