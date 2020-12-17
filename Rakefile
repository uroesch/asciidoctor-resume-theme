require 'yaml'
require 'erb'
require 'etc'
require 'ostruct'

# -----------------------------------------------------------------------------
# Global constants
# -----------------------------------------------------------------------------
BASE_DIR     = File.dirname(__FILE__)
SRC_DIR      = File.join(BASE_DIR, '_src')
FACTORY_DIR  = File.join(BASE_DIR, 'factory')
STYLES_DIR   = BASE_DIR
DOCKER_IMAGE = 'uroesch/docker-asciidoctor:latest'

# -----------------------------------------------------------------------------
# Methods
# -----------------------------------------------------------------------------
def inside_docker?
  File.exist?('/.dockerenv')
end

def docker_asciidoctor(program = 'asciidoctor', **kwargs)
  return program if inside_docker?
  workdir = kwargs.fetch(:workdir, BASE_DIR)
  docker_options = [
    '--rm',
    '--tty',
    '--interactive',
    "--mount type=bind,src=#{BASE_DIR},target=#{BASE_DIR}",
    "--workdir=#{workdir}",
    "--user #{Process.uid}:#{Process.gid}",
    DOCKER_IMAGE,
    program
  ]
  "docker run " << docker_options.join(' ')
end

def asciidoctor_pdf
  docker_asciidoctor('asciidoctor-pdf')
end

def asciidoctor
  docker_asciidoctor
end

def compass
  docker_asciidoctor('compass', workdir: FACTORY_DIR)
end

def to_filename(file)
  file.pathmap('%f')
end

def to_basename(file)
  file.pathmap('%n')
end

# -----------------------------------------------------------------------------
# Tasks
# -----------------------------------------------------------------------------
task :default => 'css:default'
task :clean => 'css:clean'

# -----------------------------------------------------------------------------
# Namespaces
# -----------------------------------------------------------------------------
namespace :css do 
  factory_url = 'https://github.com/asciidoctor/asciidoctor-stylesheet-factory.git'
  factory_dir = to_basename(FACTORY_DIR)
  sass_dir    = File.join(factory_dir, 'sass')

  desc "Clean css creation"
  task :clean do 
    sh %(git submodule deinit --force #{factory_dir})  
    sh %(git rm -rf #{factory_dir})  
  end

  task :clone_factory do
    next if File.exist?(factory_dir)   
    sh %(git submodule add --force #{factory_url} #{factory_dir})
  end

  task :clean_scss => :clone_factory do
    rm_rf sass_dir
  end

  task :copy_scss => :clean_scss do
    sh %(cp -va #{SRC_DIR}/sass #{factory_dir})
  end

  task :default => :copy_scss do
    sh %(#{compass} compile --css-dir #{STYLES_DIR} -s compressed)
  end
end
