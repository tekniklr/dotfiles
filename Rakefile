require 'rake'
require 'erb'

desc "install the dot files into user's home directory"
task :install do
  puts "Copying dotfiles..."
  Dir['files/*'].each do |file|
    lastmod = File.mtime(file)
    file.sub!('files/', '')
    determine_action(file, ENV['HOME'], true, lastmod)
  end
  puts "Copying launchagents..."
  Dir['LaunchAgents/*'].each do |agent|
    lastmod = File.mtime(agent)
    agent.sub!('LaunchAgents/', '')
    determine_action(agent, "#{ENV['HOME']}/Library/LaunchAgents", false, lastmod)
  end
end

def determine_action(file, target, hidden, lastmod)
  full_filename = File.join(target, (hidden ? ".#{file}" : "#{file}"))
  if File.exist?(full_filename.sub('.erb', ''))
    if File.mtime(full_filename.sub('.erb', '')) >= lastmod
      puts "\t#{file.sub('.erb', '')} is up to date"
    else
      print "\toverwrite #{file.sub('.erb', '')}? [ynq] "
      case $stdin.gets.chomp
      when 'y'
        replace_file(full_filename)
      when 'q'
        exit
      else
        puts "\tskipping #{file.sub('.erb', '')}"
      end
    end
  else
    puts "\t#{full_filename.sub('.erb', '')} does not exist yet!"
    link_file(full_filename)
  end
end

def replace_file(file)
  system %Q{rm -rf "#{file.sub('.erb', '')}"}
  link_file(file)
end

def link_file(file)
  base_file = file.gsub(/\A#{ENV['HOME']}(\/Library\/LaunchAgents)?\/\.?/, '')
  dotfile = "#{ENV['PWD']}/files/#{base_file}"
  launchagent = "#{ENV['PWD']}/LaunchAgents/#{base_file}"
  source_file = false
  if File.exist?(dotfile)
    source_file = dotfile
  elsif File.exist?(launchagent)
    source_file = launchagent
  end
  unless source_file
    puts "\t*** Source file could not be found!"
    return
  end
  if file =~ /.erb$/
    puts "\tgenerating #{file.sub('.erb', '')}"
    File.open("#{file.sub('.erb', '')}", 'w') do |new_file|
      new_file.write ERB.new(File.read(source_file)).result(binding)
    end
  else
    puts "\tlinking #{file} -> #{source_file}"
    system %Q{ln -s "#{source_file}" "#{file}"}
  end
end