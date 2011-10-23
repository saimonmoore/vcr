using_git = File.exist?(File.expand_path('../.git/', __FILE__))

if using_git
  require 'bundler'
  require 'bundler/setup'
  Bundler::GemHelper.install_tasks
end

require 'rake'
require "rspec/core/rake_task"

RSpec::Core::RakeTask.new(:spec) do |t|
  t.verbose = false

  # we require spec_helper so we don't get an RSpec warning about
  # examples being defined before configuration.
  t.ruby_opts = "-w -I./spec -r./spec/capture_warnings -rspec_helper"
  t.rspec_opts = %w[--format progress] if (ENV['FULL_BUILD'] || !using_git)
end

desc "Run all examples using rcov"
RSpec::Core::RakeTask.new :rcov => :cleanup_rcov_files do |t|
  t.rcov = true
  t.rcov_opts =  %[-Ilib -Ispec --exclude "spec/*,gems/*,ping,basic_object" --text-report --sort coverage --aggregate coverage.data]
end

task :cleanup_rcov_files do
  rm_rf 'coverage.data'
end

require 'cucumber/rake/task'
Cucumber::Rake::Task.new

task :default => [:spec, :cucumber]

namespace :ci do
  desc "Sets things up for a ci build on travis-ci.org"
  task :setup do
    ENV['TRAVIS'] = 'true'
    sh "git submodule init"
    sh "git submodule update"
  end

  RSpec::Core::RakeTask.new(:spec) do |t|
    t.verbose = true

    # we require spec_helper so we don't get an RSpec warning about
    # examples being defined before configuration.
    t.ruby_opts = "-w -I./spec -r./spec/capture_warnings -rspec_helper"
    t.rspec_opts = %w[--format progress --backtrace]
  end

  desc "Run a ci build"
  task :build => [:setup, :spec, :cucumber]
end

def ensure_relish_doc_symlinked(filename)
  from_filename = filename.dup
  from_filename << '.md' unless filename =~ /\.md$/
  from = File.expand_path("../features/#{from_filename}", __FILE__)
  to = File.expand_path("../#{filename}", __FILE__)

  if File.symlink?(from)
    return if File.readlink(from) == to

    # delete the old symlink
    File.unlink(from)
  end

  FileUtils.ln_s to, from
end

desc "Push cukes to relishapp using the relish-client-gem"
task :relish do
  %w[ README.md CHANGELOG.md LICENSE ].each do |file|
    ensure_relish_doc_symlinked(file)
  end

  require 'vcr/version'
  sh "relish versions:add myronmarston/vcr:#{VCR.version}" if ENV['NEW_RELISH_RELEASE']
  sh "relish push vcr:#{VCR.version}"
end

task :prep_relish_release do
  ENV['NEW_RELISH_RELEASE'] = 'true'
end

task :require_ruby_18 do
  raise "This must be run on Ruby 1.8" unless RUBY_VERSION =~ /^1\.8/
end

task :release => [:require_ruby_18, :prep_relish_release, :relish]

# For gem-test: http://gem-testers.org/
task :test => :spec

load './lib/vcr/tasks/vcr.rake'
