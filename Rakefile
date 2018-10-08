require "rake/extensiontask"
require 'rake/testtask'
require 'bundler/gem_tasks'
require 'net/scp'

Rake::ExtensionTask.new "geoip2_compat" do |ext|
  ext.lib_dir = "lib/geoip2_compat"
end

Rake::TestTask.new do |t|
  t.libs << "test"
  t.test_files = FileList['test/test*.rb']
  t.verbose = true
end

task :download do
  file = "test/GeoLite2-City.mmdb"
  unless File.exist?(file)
    `curl http://geolite.maxmind.com/download/geoip/database/GeoLite2-City.mmdb.gz | gzip -d > #{file}`
  end
end

task :vendor do
  version = "1.3.2"
  mkdir_p "tmp/"
  dir = "tmp/libmaxminddb-#{version}"
  cd "tmp/" do
    sh "curl -L https://github.com/maxmind/libmaxminddb/releases/download/#{version}/libmaxminddb-#{version}.tar.gz | tar xz"
  end
  cp "#{dir}/src/data-pool.h", "ext/geoip2_compat/data-pool.h"
  cp "#{dir}/src/data-pool.c", "ext/geoip2_compat/data-pool.c"
  cp "#{dir}/src/maxminddb-compat-util.h", "ext/geoip2_compat/maxminddb-compat-util.h"
  cp "#{dir}/src/maxminddb.c", "ext/geoip2_compat/maxminddb.c"
  cp "#{dir}/include/maxminddb.h", "ext/geoip2_compat/maxminddb.h"
  cp "#{dir}/include/maxminddb_config.h.in", "ext/geoip2_compat/maxminddb_config.h"
end

desc "upload gem and recreate index"
task :upload_gem => [:build] do
  hostname = "svn01"
  username = "hgadmin"
  gem_repo_path ="/data/gemrepo"

  Dir["pkg/*gem"].each do |file|
    puts "Upload #{file}"
    Net::SCP.start(hostname, username, :verbose => 1) do |scp|
      # synchronous (blocking) upload; call blocks until upload completes
      scp.upload! file, "#{gem_repo_path}/gems"
    end
    File.delete(file)
  end

  Net::SSH.start(hostname, username, :verbose => 1) do |ssh|
    puts ssh.exec!("gem generate_index -d #{gem_repo_path}")
  end
end

task :default => [:compile, :download, :test]
