# rcovrt4j: This builds a JRuby extension for Rcov. 
# 
# Compiles java and makes rcov service jar. It will
# also package it as a gem for release into the wild.
require 'rake'
require 'rubygems'
require 'rake/gempackagetask'
require 'rake/testtask'
require 'rake/rdoctask'

GEM_NAME='ruby-debug-base'
GEM_VERSION='0.9.3'

def java_classpath_arg
  begin
    require 'java'
    classpath = java.lang.System.getProperty('java.class.path')
  rescue LoadError
  end

  unless classpath
    classpath = FileList["#{ENV['JRUBY_HOME']}/lib/*.jar"].join(File::PATH_SEPARATOR)
  end

  classpath ? "-cp #{classpath}" : ""
end

def compile_java
  mkdir_p "java/classes"
  sh "javac -g -target 1.5 -source 1.5 -d java/classes #{java_classpath_arg} #{FileList['src/**/*.java'].join(' ')}"
end

def make_jar
  require 'fileutils'
  lib = File.join(File.dirname(__FILE__), 'lib')
  FileUtils.mkdir(lib) unless File.exists? lib
  sh "jar cf lib/ruby_debug_base.jar -C java/classes/ ."
end

file 'lib/ruby_debug_base.jar' => FileList["java/src/*.java"] do
  compile_java
  make_jar
end

spec = Gem::Specification.new do |s|
  s.platform = "java"
  s.summary  = "Java implementation of Fast Ruby Debugger"
  s.name     = GEM_NAME
  s.version  = GEM_VERSION
  s.require_path = 'lib'
  s.files    = FileList['Rakefile', 'README', 'lib/ruby_debug_base.jar', 'lib/ruby-debug-base.rb']
  s.description = "Java extension to make fast ruby debugger run on JRuby"
  #s.add_dependency 'rcov', '>= 0.8.0.2'
  s.author   = 'Chris Nelson'
  s.email    = 'superchrisnelson@gmail.com'
  s.homepage = 'http://rubyforge.org/projects/commons-debugger'
  s.has_rdoc = true
end

Rake::GemPackageTask.new(spec).define

Rake::RDocTask.new do |t|
  t.main = 'README'
  t.rdoc_files.include 'README'
end

desc "Install the gem file #{GEM_NAME}-#{GEM_VERSION}-java.gem"
task :install_gem do
  gem = File.join(File.dirname(__FILE__), 'pkg', "#{GEM_NAME}-#{GEM_VERSION}-java.gem")
  ruby '-S', 'gem', 'install', gem
end

desc "Uninstall the gem #{GEM_NAME}"
task :uninstall_gem do
  ruby '-S', 'gem', 'uninstall', GEM_NAME
end

desc "Create the Java extension."
task :compile => ['lib/ruby_debug_base.jar']
task :gem => [:compile]
task :install_gem => [:gem]
