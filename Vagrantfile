# -*- mode: ruby -*-
# vi: set ft=ruby :
require 'json'

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"
config_file = ENV["config"] || './config.json'

if not File.exists? config_file
	abort "ERROR: Missing configuration file: #{config_file}"
end

setup = JSON.load(IO.read(config_file));

def bootstrap_args (setup)
	args = []
	return args if setup.to_s == ''

	args.push('-n', setup["git"]["user"]["name"])
	args.push('-e', setup["git"]["user"]["email"])
	if setup["git"].has_key? "version"
		args.push('-v', setup["git"]["version"])
	end

	if setup.has_key? "npm" and setup["npm"].has_key? "useSystemProxy" and setup["npm"]["useSystemProxy"] and ENV.has_key? "http_proxy"	
		args.push('-p', ENV['http_proxy']) 
	else
		args.push('-p', 'null') 
	end

	args.push('-r', setup["npm"]["registry"]) if setup.has_key? "npm" and setup["npm"].has_key? "registry"
	args
end

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
	config.vm.hostname    = "devenv"
	config.vm.box         = "chef/ubuntu-14.04"
	config.vm.network       "private_network", ip: "192.168.50.2"
	if setup.has_key? 'syncedFolders'
		setup["syncedFolders"].each do |sync|
			config.vm.synced_folder sync["source"], sync["dest"]
		end
	end
	config.ssh.forward_agent = true

	config.vm.provider :virtualbox do |vb|
		vb.name = "devenv"
		vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
		vb.customize ["modifyvm", :id, "--memory", "4096"]
	end

	if Vagrant.has_plugin?("vagrant-hosts") and setup.has_key? "hosts"
		config.vm.provision :hosts do |hosts|
			setup["hosts"].each do |host|
				hosts.add_host host["ip"], host["names"]
			end
		end
	end

	if Vagrant.has_plugin?("vagrant-proxyconf") and setup.has_key? "proxy" and setup["proxy"].has_key? "useSystemProxy" and setup["proxy"]["useSystemProxy"] and ENV.has_key? "http_proxy"
		config.proxy.http     = ENV['http_proxy']
		config.proxy.https    = ENV['https_proxy']
		config.proxy.no_proxy = ENV['no_proxy']
	end

	config.vm.provision :file, source: "./home/.vimrc", destination: "~/.vimrc"
	config.vm.provision :file, source: "./home/.gemrc", destination: "~/.gemrc"
	config.vm.provision :file, source: "./home/.tmux.conf", destination: "~/.tmux.conf"
	config.vm.provision :file, source: "./home/.gitconfig", destination: "~/.gitconfig"

	config.vm.provision :ventriloquist do |env|
		env.packages << %w( tmux build-essential libssl-dev libcurl4-gnutls-dev libexpat1-dev gettext libz-dev checkinstall exuberant-ctags curl vim-nox cmake dstat gnuplot gdb unzip autoconf automake libtool htop )
	end

	args = bootstrap_args setup
	config.vm.provision :shell, :path => "bootstrap.sh", :args => args
end
