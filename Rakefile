#!/usr/bin/env rake
require 'yaml'

DEFAULT_RUBIES = %w{2.1.1}.freeze

module Rake
  module FileUtilsExt
    alias :real_rake_output_message :rake_output_message
    def rake_output_message(message)
      real_rake_output_message(filter_secrets(message))
    end

    private

    SECRETS = %w{AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY}

    def filter_secrets(message)
      message.gsub(%r{(#{SECRETS.join('|')})=('?[^\s']*'?)}, '\1=<protected>')
    end
  end
end

def vagrant(box, action = "up")

  rubies = @config['rbenv_rubies'] || DEFAULT_RUBIES

  cmd = [
    %{VAGRANT_BOX='#{box}'},
    %{AWS_ACCESS_KEY_ID='#{@config['aws_access_key_id']}'},
    %{AWS_SECRET_ACCESS_KEY='#{@config['aws_secret_access_key']}'},
    %{S3_BUCKET='#{@config['s3_bucket']}'},
    %{RBENV_RUBIES="#{rubies.join(' ')}"},
    %{vagrant #{action}}
  ].join(' ')
  sh "time (#{cmd})"
end

desc "Prepares project space"
task :init do
  sh "librarian-chef install --verbose"
  cp "config.sample.yml", "config.yml" unless File.exists?("config.yml")
  @config = YAML::load_file(File.expand_path('./config.yml'))
end

desc "Builds and uploads Ruby versions one box"
task :build => [:init] do
  abort "\n\nVAGRANT_BOX must be set, aborting." unless ENV['VAGRANT_BOX']

  box     = ENV['VAGRANT_BOX']
  action  = ENV['VAGRANT_ACTION'] || "up"

  vagrant box, action
  vagrant box, "destroy -f"
end

[:up, :halt, :provision, :reload, :status, :ssh, :destroy].each do |action|
  desc "Runs vagrant #{action.to_s}"
  task action => [:init] do
    abort "\n\nVAGRANT_BOX must be set, aborting." unless ENV['VAGRANT_BOX']
    box = ENV['VAGRANT_BOX']
    vagrant box, action.to_s
  end
end

desc "Cleans up temporary files/directories"
task :cleanup do
  rm_rf %w{Cheffile.lock cookbooks tmp}
end

desc "Builds and uploads Ruby versions for all boxes"
task :build_all => [:cleanup, :init] do
  Array(@config['vagrant_boxes']).each do |box|
    vagrant box, "up"
    vagrant box, "destroy -f"
  end
end

task :default => 'init'
