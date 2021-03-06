require 'rubygems'
require 'rake'
require 'rake/testtask'
require 'rake/packagetask'
require 'rake/gempackagetask'

gemfile = File.expand_path('../spec/test_app/Gemfile', __FILE__)
if File.exists?(gemfile) && (%w(spec cucumber).include?(ARGV.first.to_s) || ARGV.size == 0)
  require 'bundler'
  ENV['BUNDLE_GEMFILE'] = gemfile
  Bundler.setup

  require 'rspec'
  require 'rspec/core/rake_task'
  RSpec::Core::RakeTask.new

  require 'cucumber/rake/task'
  Cucumber::Rake::Task.new do |t|
    t.cucumber_opts = %w{--format progress}
  end
end


desc "Default Task"
task :default => [ :spec ]
# task :default => [:spec, :cucumber ]

spec = eval(File.read('spree_additional_calculators.gemspec'))

Rake::GemPackageTask.new(spec) do |p|
  p.gem_spec = spec
end


desc "Release to gemcutter"
task :release => :package do
  require 'rake/gemcutter'
  Rake::Gemcutter::Tasks.new(spec).define
  Rake::Task['gem:push'].invoke
end


desc "Regenerates a rails 3 app for testing"
task :test_app do
  SPREE_PATH = ENV['SPREE_PATH']
  raise "SPREE_PATH should be specified" unless SPREE_PATH
  require File.join(SPREE_PATH, 'lib/generators/spree/test_app_generator')
  class SpreeAdditionalCalculatorTestAppGenerator < Spree::Generators::TestAppGenerator

    def tweak_gemfile
      append_file 'Gemfile' do
<<-gems
gem 'spree_core', :path => '#{File.join(SPREE_PATH, 'core')}'
gem 'spree_auth', :path => '#{File.join(SPREE_PATH, 'auth')}'
gem 'spree_additional_calculators', :path => '#{File.dirname(__FILE__)}'
gems
      end
    end

    def install_gems
      inside "test_app" do
        run 'rake spree_core:install'
        run 'rake spree_auth:install'
        run 'rake spree_additional_calculators:install'
      end
    end

    def migrate_db
      run_migrations
    end
  end
  SpreeAdditionalCalculatorTestAppGenerator.start
end


namespace :test_app do
  desc 'Rebuild test and cucumber databases'
  task :rebuild_dbs do
    system("cd spec/test_app && rake db:drop db:migrate RAILS_ENV=test && rake db:drop db:migrate RAILS_ENV=cucumber")
  end
end
