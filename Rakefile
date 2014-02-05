# encoding: UTF-8

require 'fileutils'
require 'erb'

@distros = %w{debian ubuntu}
@targets = %w{virtualbox vmware}
@types = %w{standard chef puppet}

namespace :build do
  @distros.each do |distro|
    FileList["#{distro}/*.erb"].each do |template_path|
      template = IO.read(template_path)
      template_name = template_path.gsub(/#{distro}\/|\.erb\z/, '')

      @targets.each do |target|
        @types.each do |type|
          name = "#{template_name}-#{type}-#{target}"
          desc "Build #{name}"
          task name do
            # Fill in the variables needed by the ERB templates.
            @type = type
            @builder =
              if target == 'virtualbox'
                'virtualbox-iso'
              else
                target
              end

            packer_template = ERB.new(template, nil, '-')
            
            # ensure tmp/ exists
            FileUtils.mkdir_p('tmp')

            # write the template to tmp
            File.open("tmp/#{name}.json", 'w') do |f|
              f.puts packer_template.result
            end

            # run packer
            result = system "packer build tmp/#{name}.json"
            
            if result
              output_name = "packer_#{template_name}-#{type}_#{target}.box" 
              FileUtils.mv(output_name, "#{name}.box")
            end

            # clean up
            FileUtils.rm("tmp/#{name}.json")
          end

          task :default => name
        end
      end
    end
  end
end

desc 'Clean up any temporary directories and files'
task :clean do
  FileUtils.rm_rf('tmp')
end

desc 'Build all of the configurations'
task :default => 'build:default'
