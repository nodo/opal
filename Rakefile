require 'bundler/gem_tasks'
require 'opal'
require 'opal/bundle_task'
require 'fileutils'

SPEC_DIR = File.join(File.dirname(__FILE__), 'spec')

header = <<-HEAD
/*!
 * opal v#{Opal::VERSION}
 * http://opalscript.org
 *
 * Copyright 2011, Adam Beynon
 * Released under the MIT license
 */
HEAD

Opal::BundleTask.new(:opal2)

desc "Rebuild core opal runtime into build/"
task :opal do
  FileUtils.mkdir_p 'build'
  parser = Opal::Parser.new
  code   = []
  order  = File.read('corelib/load_order').strip.split
  core   = order.map { |c| File.read("corelib/#{c}.rb") }

  %w[pre runtime init class module fs loader vm id].each do |r|
    code << File.read("runtime/#{r}.js")
  end

  # runtime
  parsed = parser.parse core.join
  code << "var core_lib = #{ parser.wrap_with_runtime_helpers(parsed[:code]) };"
  code << File.read("runtime/post.js")

  # methods
  File.open('build/data.yml', 'w+') do |f|
    f.write({"methods"  => parsed[:methods],
            "ivars"     => parsed[:ivars],
            "next"      => parsed[:next]}.to_yaml)
  end

  # boot - bare code to be used in output projects
  File.open('build/opal.js', 'w+') do |f|
    f.write header
    f.write code.join
  end
end

desc "Run opal tests"
task :test => :opal do
  if glob = ENV['TEST']
    glob = File.expand_path glob, SPEC_DIR
    glob += "/**/*.rb" if File.directory? glob
  end

  src = Dir[glob || 'spec/**/*.rb']

  abort "no matching tests for #{glob.inspect}" if src.empty?

  c = Opal::Context.new
  src.each { |s| c.eval File.read(s), s}
  c.finish
end

desc "Check file sizes for core builds"
task :sizes do
  o = File.read "build/opal.js"
  m = uglify(o)
  g = gzip(m)

  File.open('vendor/opal.min.js', 'w+') { |o| o.write m }

  puts "development: #{o.size}, minified: #{m.size}, gzipped: #{g.size}"
end

# Used for uglifying source to minify
def uglify(str)
  IO.popen('uglifyjs -nc', 'r+') do |i|
    i.puts str
    i.close_write
    return i.read
  end
end

# Gzip code to check file size
def gzip(str)
  IO.popen('gzip -f', 'r+') do |i|
    i.puts str
    i.close_write
    return i.read
  end
end

task :docs => "docs:build"

namespace :docs do
  task :build => :build_js do
    Dir.chdir("docs") { system "jekyll" }
    %w[opal.js opal-parser.js].each do |s|
      FileUtils.cp s, "docs/_site/#{s}", :verbose => true
    end
  end

  task :publish do
    if File.exist? "gh-pages"
      puts "./gh-pages already exists, so skipping clone"
    else
      sh "git clone -b gh-pages git@github.com:adambeynon/opal.git gh-pages"
    end
    FileUtils.cp_r "docs/_site/.", "gh-pages", :verbose => true
  end
end

