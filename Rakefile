require 'rake'
require 'rubygems'
require 'rake/clean'
require 'rake/gempackagetask'
require 'rake/testtask'
require 'rake/rdoctask'

GEM_NAME='ruby-debug-base'
GEM_VERSION='0.10.1'

RUBY_DEBUG_JAR='ext/ruby_debug.jar'

CLEAN.include('ext')
CLEAN.include('lib/ruby_debug.jar')

DIST_FILES = FileList[
  'AUTHORS',
  'ChangeLog',
  'lib/linecache.rb',
  'lib/linecache-ruby.rb',
  'lib/ruby-debug-base.rb',
  'lib/ruby_debug.jar',
  'lib/tracelines.rb',
  'MIT-LICENSE',
  'Rakefile',
  'README'
]

BASE_TEST_FILE_LIST = %w(
  test/base/base.rb
  test/base/binding.rb 
  test/base/catchpoint.rb)

task :default => :package

CLI_TEST_FILE_LIST = 'test/test-*.rb'

desc "Test ruby-debug-base."
task :test_base => :lib do 
  Rake::TestTask.new(:test_base) do |t|
    t.libs << ['./ext', './lib']
    t.test_files = FileList[BASE_TEST_FILE_LIST]
    t.verbose = true
  end
end

desc "Test everything."
task :test => :test_base do 
  Rake::TestTask.new(:test) do |t|
    t.libs << ['./ext', './lib', './cli']
    t.pattern = CLI_TEST_FILE_LIST
    t.verbose = true
  end
end

desc "Helps to setup the project to be able to run tests"
task :prepare_tests do
  # needed to run CLI test. Unable to use svn:externals yet:
  #   http://subversion.tigris.org/issues/show_bug.cgi?id=937
  sh 'svn cat svn://rubyforge.org/var/svn/ruby-debug/tags/ruby-debug-0.10.1/rdbg.rb > rdbg.rb' unless File.exists?('rdbg.rb')

  # prepare default customized test/config.private.yaml suitable for JRuby
  File.open('test/config.private.yaml', 'w') do |f| 
    f.write(<<DOC
# either should be on the $PATH or use full path
ruby: jruby

# possibility to specify interpreter parameters
ruby_params: -J-Djruby.reflection=true -J-Djruby.compile.mode=OFF
DOC
    )
  end unless File.exists?('test/config.private.yaml')
end

desc "Create the core ruby-debug shared library extension"
task :lib do
  compile_java
  make_jar
end

file RUBY_DEBUG_JAR => :lib

spec = Gem::Specification.new do |s|
  s.platform = "java"
  s.summary  = "Java implementation of Fast Ruby Debugger"
  s.name     = GEM_NAME
  s.version  = GEM_VERSION
  s.require_path = 'lib'
  s.files    = DIST_FILES
  s.description = <<-EOF
Java extension to make fast ruby debugger run on JRuby.
It is the same what ruby-debug-base is for native Ruby.
EOF
  s.author   = 'debug-commons team'
  s.homepage = 'http://rubyforge.org/projects/debug-commons/'
  s.has_rdoc = true
  s.rubyforge_project = 'debug-commons'
end

gem_name = "#{GEM_NAME}-#{GEM_VERSION}-#{spec.platform}.gem"

desc "Build the gem file #{gem_name}"
task :gem => :lib do
  gem_task = Rake::GemPackageTask.new(spec)
  current_dir = File.expand_path(File.dirname(__FILE__))
  source = File.join(current_dir, RUBY_DEBUG_JAR)
  target = File.join(current_dir, "lib", "ruby_debug.jar")
  cp(source, target)
  # Create the gem, then move it to pkg.
  Gem::Builder.new(spec).build
  gem_file = "#{spec.name}-#{spec.version}-#{spec.platform}.gem"
  mv(gem_task.gem_file, "pkg/#{gem_task.gem_file}")
  # Delete the temporary target
  rm target
end

desc 'Build all the packages'
task :package => :gem

Rake::RDocTask.new do |t|
  t.main = 'README'
  t.rdoc_files.include 'README'
end


desc "Create a GNU-style ChangeLog via svn2cl"
task :ChangeLog do
  system("svn2cl --authors=svn2cl_usermap svn://rubyforge.org/var/svn/debug-commons/jruby-debug/trunk")
end

# Builds classpath containing needed JRuby's jars (like jruby.jar).
def jruby_classpath
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

# Compiles Java classes into the pkg/classes directory.
def compile_java
  mkdir_p "pkg/classes"
  sh "javac -Xlint -Xlint:-serial -g -target 1.5 -source 1.5 -d pkg/classes #{jruby_classpath} #{FileList['src/**/*.java'].join(' ')}"
end

def make_jar
  require 'fileutils'
  ext = File.join(File.dirname(__FILE__), 'ext')
  FileUtils.mkdir_p(ext)
  separator = File::ALT_SEPARATOR || File::SEPARATOR
  sh "jar cf #{RUBY_DEBUG_JAR} -C pkg#{separator}classes ."
end

