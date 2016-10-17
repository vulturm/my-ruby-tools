#!/usr/bin/env rake
# encoding: utf-8
#
# Rakefile helper
#
# Copyright (C) 2016 Mihai Vultur
#
# All rights reserved - Do Not Redistribute
#

require 'rake'
require 'rspec/core/rake_task'
#--
desc 'Run all tests except Kitchen (default task)'
task default: 'lint'
#--
desc 'Run linters'
multitask lint: %w(linters:rubocop linters:foodcritic linters:chefspec)
#--
desc 'Run all tests, including kitchen'
task test: %w(lint integration:kitchentest)
#--
desc 'Need to export REMOTE_HOST env'
task remote: 'validation:spec'

# Style tests. cookstyle (rubocop) and Foodcritic
namespace :linters do
  #--
  begin
    require 'rubocop/rake_task'
    desc 'Run rubocop style checks'
    RuboCop::RakeTask.new(:rubocop) do |t|
      t.fail_on_error = true
    end
  rescue StandardError => e
    puts ">>> Gem load error: #{e}, omitting style:ruby" unless ENV['CI']
  end
  #--
  begin
    require 'foodcritic'
    desc 'Run Chef style checks'
    FoodCritic::Rake::LintTask.new(:foodcritic) do |t|
      t.options = {
        fail_tags: ['any'],
        progress: true
      }
    end
  rescue LoadError => e
    puts ">>> Gem load error: #{e}, omitting style:chef" unless ENV['CI']
  end

  desc 'Run Chefspec tests'
  RSpec::Core::RakeTask.new(:chefspec)
end

#-- Integration tests. Kitchen.ci
namespace :integration do
  begin
    require 'kitchen/rake_tasks'
    desc 'Run kitchen integration tests'
    Kitchen::RakeTasks.new
    task kitchentest: ['kitchen:all']
  rescue StandardError => e
    puts ">>> Kitchen error: #{e}, omitting #{task.name}" unless ENV['CI']
  end
end

#-- remote converged host validation
namespace :validation do
  begin
    target = ENV['REMOTE_HOST'] || 'kitchen'
    next if target == 'kitchen'
    puts "Run serverspec tests on remote: #{target}"
    RSpec::Core::RakeTask.new do |t|
      t.pattern = 'test/integration/*/**/*_spec.rb'
      t.rspec_opts = ['-I test/integration/helpers/serverspec', '--format documentation', '--color'] # , '--fail-fast' ]
      t.verbose = true
    end
  rescue StandardError => e
    puts ">>> Remote validation error: #{e}, omitting #{task.name}" unless ENV['CI']
  end
end
