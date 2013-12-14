require 'bundler/gem_tasks'
require 'bundler/setup'
require 'ffi/gen'
require 'rake/testtask'
require 'yard'

task :default => :test

task :check do
  sh 'git submodule update --init' unless File.exist?('ext/rocksdb/.git')
end

desc "Generates rocksdb ext"
task :compile => :check do
  sh 'cd ext && rake'
end

desc "Clean rocksdb build"
task :clean do
  sh 'cd ext/rocksdb && make clean'
end

task :release => :clean

desc "Rebuild rocksdb"
task :rebuild => [:clean, :compile]

default_config = -> (header, suffix="", extra={}) do
  {
    module_name: 'Rocksdb',
    ffi_lib:     'File.expand_path("../../ext/rocksdb/librocksdb.#{FFI::Platform::LIBSUFFIX}", __FILE__)',
    headers:     [File.expand_path("../ext/rocksdb/include/rocksdb/#{header}", __FILE__)],
    cflags:      `llvm-config --cflags`.split(" "),
    prefixes:    ['rocksdb_'],
    suffixes:    ['_t', '_s'],
    output:      "lib/native#{suffix}.rb"
  }.merge(extra)
end

Rake::TestTask.new do |t|
  t.libs << 'test'
  t.test_files = FileList['test/test_*.rb']
end

desc 'Generates FFI bindings'
task :ffi => :check do
  raise 'Please install llvm' unless system('which llvm-config')
  FFI::Gen.generate(default_config["c.h"])
end

desc 'Open a console'
task :console do
  ARGV.clear
  require 'irb'
  require 'leveldb'
  IRB.start
end

YARD::Rake::YardocTask.new(:doc) do |t|
  t.files = ['lib/**/*.rb']
  t.options = ['--markup=markdown',
               '--no-private',
               '--markup-provider=redcarpet']
end
