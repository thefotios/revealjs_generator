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

# These are possible values for config that can be overidden by ENV vars
#   If you use a new config value, make sure to add it here
DEFAULT_CONFIG = {
  :author => nil,
  :description => nil,
  :title => nil,
  :theme => nil,
  :out => 'out'
}

task :render do
  # Copy reveal.js
  dest = config.out
  FileUtils.rm_r(dest) if Dir.exists?(dest)
  FileUtils.cp_r 'reveal.js', dest
  FileUtils.rm Dir.glob("#{dest}/.git*")

  # Load the templates
  @doc = load_template
  # Modify the values
  update_doc("//meta[@name=author]",config.author,"content")
  update_doc("//meta[@name=description]",config.description.strip,"content")
  update_doc("//title",config.title)
  update_doc("//link#theme",("css/theme/%s.css" % config.theme),"href") if config.theme

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
end

task :publish do
  dest = config.out
  if Dir.exists?(dest)
    gh_pages(dest)
  else
    abort("Rendered directory does not exist: %s. Run `rake render` first" % dest)
  end
end

def gh_pages(dir)
  `git add #{dir}`
  sha = `git write-tree`.chomp
  tree_sha = `git rev-parse #{sha}:#{dir}`.chomp
  `git read-tree HEAD`  # reset staging to last-commit
  ghp_sha = `git rev-parse gh-pages 2>/dev/null`.chomp
  extra = ghp_sha != 'gh-pages' ? "-p #{ghp_sha}" : ''
  commit_sha = `echo 'static presentation' | git commit-tree #{tree_sha} #{extra}`.chomp
  `git update-ref refs/heads/gh-pages #{commit_sha}`
  `git push origin gh-pages -f`
end

def git_commit(path = nil)
  `git add #{path}`
  `git commit -m "made commit"`
  rev_parse(path)
end

def rev_parse(rev = nil)
  commit = "HEAD"
  commit << ":#{rev}" if rev
  `git rev-parse #{commit}`.chomp
end

def config
  @config ||= parse_config
end

def parse_config
  puts "Parsing configuration"
  # Parse the config file
  config = YAML.load_file('config.yml')
  # Get any env variables that also appear in the default config
  env = Hash[ENV.select{|k,v| DEFAULT_CONFIG.has_key?(k.to_sym)}.map{|k,v| [k.to_sym,v]}]
  # Merge them all together
  merged = [DEFAULT_CONFIG,config,env].reduce({},:merge)
  OpenStruct.new(merged)
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
