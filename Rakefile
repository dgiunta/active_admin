require "bundler"
Bundler.setup
Bundler::GemHelper.install_tasks

require 'rake'

def cmd(command)
  puts command
  system command
end

desc "Install all supported versions of rails"
task :install_all_rails do
  (0..7).to_a.each do |v|
    system "rm Gemfile.lock"
    puts "Installing for RAILS=3.0.#{v}"
    cmd "RAILS=3.0.#{v} bundle install"
  end
end

desc "Creates a test rails app for the specs to run against"
task :setup do
  require 'rails/version'
  system("mkdir spec/rails") unless File.exists?("spec/rails")
  system "bundle exec rails new spec/rails/rails-#{Rails::VERSION::STRING} -m spec/support/rails_template.rb"
end

namespace :local do
  desc "Creates a local rails app to play with in development"
  task :setup do
    unless File.exists? "test-rails-app"
      system "bundle exec rails new test-rails-app -m spec/support/rails_template_with_data.rb"
    end
  end

  desc "Start a local rails app to play"
  task :server => :setup do
    exec "cd test-rails-app && GEMFILE=../Gemfile bundle exec rails s"
  end

  desc "Load the local rails console"
  task :console => :setup do
    exec "cd test-rails-app && GEMFILE=../Gemfile bundle exec rails c"
  end
end

require "rspec/core/rake_task"
RSpec::Core::RakeTask.new(:spec)

RSpec::Core::RakeTask.new(:rcov) do |spec|
  spec.rcov = true
end

# Run the specs & cukes
task :default do
  # Force spec files to be loaded and ran in alphabetical order.
  specs_unit = Dir['spec/unit/**/*_spec.rb'].sort.join(' ')
  specs_integration = Dir['spec/integration/**/*_spec.rb'].sort.join(' ')
  exit [
    cmd("export RAILS=3.0.5 && export RAILS_ENV=test && bundle exec rspec #{specs_unit}"),
    cmd("export RAILS=3.0.5 && export RAILS_ENV=test && bundle exec rspec #{specs_integration}"),
    cmd("export RAILS=3.0.5 && export RAILS_ENV=cucumber && bundle exec cucumber features"),
  ].uniq == [true]
end

namespace :spec do
  desc "Run specs for all versions of rails"
  task :all do
    (0..6).to_a.each do |v|
      puts "Running for Rails 3.0.#{v}"
      cmd "rm Gemfile.lock" if File.exists?("Gemfile.lock")
      cmd "/usr/bin/env RAILS=3.0.#{v} bundle install"
      cmd "/usr/bin/env RAILS=3.0.#{v} rake spec"
    end
  end
end

require 'cucumber/rake/task'

namespace :cucumber do
  Cucumber::Rake::Task.new(:all) do |t|
    t.profile = 'default'
  end
  
  Cucumber::Rake::Task.new(:wip) do |t|
    t.profile = 'wip'
  end
end

task :cucumber => "cucumber:all"

require 'rake/rdoctask'
Rake::RDocTask.new do |rdoc|
  version = File.exist?('VERSION') ? File.read('VERSION') : ""

  rdoc.rdoc_dir = 'rdoc'
  rdoc.title = "active_admin #{version}"
  rdoc.rdoc_files.include('README*')
  rdoc.rdoc_files.include('lib/**/*.rb')
end
