require 'rubygems'
require 'fileutils'
require 'erb'

CONFIG = {
  :project_root => File.dirname(__FILE__),
  :build => "build",
  :cassandra => {
    :home             => File.dirname(__FILE__) + "/build/apache-cassandra-1.0.8",
    :data_dir         => File.dirname(__FILE__) + "/data",
    :cluster_name     => 'test cassandra cluster',
    :initial_token    => '0',
    :listen_address   => 'localhost',
    :storage_port     => '7000',
    :ssl_storage_port => '7001',
    :jmx_port         => '7199',
    :rpc_port         => '9160'
  },
  :urls => {
    :package => {
      :url => "http://apache.mirrors.pair.com/cassandra/1.0.8/apache-cassandra-1.0.8-bin.tar.gz",
      :file => "apache-cassandra-1.0.8-bin.tar.gz"
    }
  }
}

CONFIG[:build]                = CONFIG[:project_root] + "/build"
CONFIG[:cassandra][:data_dir] = CONFIG[:build]        + "/data"

def platform
  @platform ||= `uname`.split.first
  case @platform
  when "Darwin"
    :osx
  else
    :linux
  end
end

def ensure_dir p
  unless File.exist? p
    FileUtils.mkdir_p p
  end
end

def in_dir p
  ensure_dir p
  Dir.chdir(p) do |p|
    yield p
  end
end

def download_cassandra url, local_file
  unless File.exist? local_file
    puts "Download: #{url} => #{local_file}"
    system "wget '#{url}'"
  end
end

def untar f
  archive_dir = `tar tzf #{CONFIG[:urls][:package][:file]} | head -n1`
  archive_dir = archive_dir.split('/').first
  puts "CASSANDRA: HOME: #{CONFIG[:cassandra][:home]}"
  unless File.exist? archive_dir
    system "tar", "xzf", f
  end
end

def render_erb src, dst, params
  bklass = Class.new do
    attr_accessor :config
  end
  b = bklass.new
  b.config = params
  result = ERB.new(File.read(src)).result(b.instance_eval { binding })
  File.open(dst,"w") do |f|
    f.write result
  end
end

namespace :cassandra do
  desc "Install"
  task :install do
    in_dir(CONFIG[:build]) do |p|
      c = CONFIG[:urls][:package]
      download_cassandra c[:url], c[:file]
      untar c[:file]
    end
    render_erb 'templates/cassandra/cassandra.yaml.erb', "#{CONFIG[:cassandra][:home]}/conf/cassandra.yaml", CONFIG[:cassandra]
    %w[data commitlog saved_cached].each do |p|
      p = "#{CONFIG[:cassandra][:data_dir]}/#{p}"
      unless File.exist? p
        FileUtils.mkdir_p p
      end
    end
    render_erb 'templates/cassandra/log4j-server.properties.erb', "#{CONFIG[:cassandra][:home]}/conf/log4j-server.properties", CONFIG[:cassandra]
    render_erb 'templates/cassandra/cassandra-env.sh.erb', "#{CONFIG[:cassandra][:home]}/conf/cassandra-env.sh", CONFIG[:cassandra]
  end

  desc "Run the server (foreground)"
  task :server do
    cmd = "#{CONFIG[:cassandra][:home]}/bin/cassandra -f"
    puts cmd
    system cmd
  end

  desc "Run the shell"
  task :shell do
    host = CONFIG[:cassandra][:listen_address]
    port = CONFIG[:cassandra][:rpc_port]
    cmd = "#{CONFIG[:cassandra][:home]}/bin/cassandra-cli -h #{host} -p #{port}"
    puts cmd
    system cmd
  end
  
end

# task :default => ["namespace::task_name"]
