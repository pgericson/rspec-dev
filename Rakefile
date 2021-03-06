require 'rubygems'
gem 'jeweler', '>= 1.4.0'
require 'rake'
require 'fileutils'
require 'pathname'

Projects = ['rspec-core', 'rspec-expectations', 'rspec-mocks', 'rspec', 'rspec-rails']
BaseRspecPath = Pathname.new(Dir.pwd)
ReposPath = BaseRspecPath.join('repos')

def run_command(command)
  Projects.each do |dir|
    path = ReposPath.join(dir)
    FileUtils.cd(path) do
      puts "="*50
      puts "# " + path.sub(/#{File.dirname(__FILE__)}\//,'')
      puts "# " + command
      puts "-"*40
      system command
      puts
    end
  end
end

task :make_repos_directory do
  FileUtils.mkdir_p ReposPath
end

def build_version_string(project_name, major, minor, tiny, pre)
version = <<versionstring
module Rspec # :nodoc:
  module #{project_name.capitalize} # :nodoc:
    module Version # :nodoc:
      unless defined?(MAJOR)
        MAJOR  = #{major}
        MINOR  = #{minor}
        TINY   = #{tiny}
        PRE    = '#{pre}'

        STRING = [MAJOR, MINOR, TINY, PRE].compact.join('.')

        SUMMARY = "#{project_name.downcase} " + STRING
      end
    end
  end
end
versionstring
end

namespace :gem do
  desc "Write out a new version constant for each project.  You must supply MAJOR, MINOR, TINY, and PRE."
  task :write_version do
    major, minor, tiny, pre = ENV['MAJOR'], ENV['MINOR'], ENV['TINY'], ENV['PRE']
    raise("You must supply MAJOR, MINOR, TINY, and PRE versions") if [major, minor, tiny, pre].any? { |v| v.nil? } 
    Projects.each do |project|
      version_string = build_version_string(project, major, minor, tiny, pre) 
      file = "repos/#{project}/lib/rspec/#{project}/version.rb"
      FileUtils.rm_rf file
      File.open(file, "w+") { |f| f << version_string }
      puts "Writing out version #{major}.#{minor}.#{tiny}.#{pre} for #{project}"
    end
  end

  desc "Rebuild gemspecs"
  task :spec do
    run_command "rake gemspec"
  end

  desc "Build gems"
  task :build => [:clean_pkg_directories] do
    run_command "rake build"
  end

  task :clean_pkg_directories do
    run_command "rm -rf pkg"
  end

  desc "Install all gems locally"
  task :install do
    run_command "rake install"
  end

  desc "Uninstall gems locally"
  task :uninstall do
    Projects.each do |project|
      path = ReposPath.join(project)
      FileUtils.cd(path) do
        system "gem uninstall --all --executables --ignore-dependencies #{project}" 
      end
    end
  end
end

namespace :dev do
  desc "Pair dev, you must supply the PAIR1, PAIR2 arguments"
  task :pair do
    raise("You must supply PAIR1, and PAIR2 to pair dev") unless ENV['PAIR1'] && ENV['PAIR2']
    run_command "pair #{ENV['PAIR1']} #{ENV['PAIR2']}"
  end

  desc "Solo dev, removes any git pair markers"
  task :solo do
    run_command "pair"
  end
end

namespace :git do
  { :status => nil, :pull => '--rebase', :push => nil, :reset => '--hard', :diff => nil }.each do |command, options|
    desc "git #{command} on all the repos"
    task command => :clone do
      run_command "git #{command} #{options}".strip
    end
  end

  task :st => :status
  task :update => :pull

  desc "git clone all the repos the first time"
  task :clone => :make_repos_directory do
    url_prefix = `git config --get remote.origin.url`[%r{(^.*)/rspec-dev}, 1]

    FileUtils.cd(ReposPath) do
      Projects.each do |repo|
        unless File.exists?(repo)
          system "git clone #{url_prefix}/#{repo}"
        end
      end
    end
  end

  desc "git commit all the repos with the same commit message"
  task :commit do
    raise("You must supply a MESSAGE to commit") unless ENV['MESSAGE']
    run_command "git commit -am '#{ENV['MESSAGE']}'"
  end
end

task :clobber do
  run_command "rake clobber"
end

task :spec do
  run_command 'rake'
end

task :gemspec do
  run_command 'rake gemspec'
end

task :default => ['git:clone', 'gem:install', :spec]
