
require 'enumerator'
require 'berkshelf/vagrant'

def configure_virtualbox_provider(config, node_name)
  config.vm.provider "virtualbox" do |v|

    v.name = node_name
    if ENV['VAGRANT_MEMORY']
      v.customize ["modifyvm", :id, "--memory", ENV['VAGRANT_MEMORY']]
    end
  end
end


def configure_chef_solo_provisioning(config, node_name, node_file_name, chef_log_level)

  vagrant_json = JSON.parse(File.open(node_file_name).read)

  config.vm.provision :chef_solo do |chef|

    chef.log_level = chef_log_level

    chef.roles_path = "roles"
    chef.data_bags_path = "data_bags"
    chef.provisioning_path = "/tmp/vagrant-chef"

    # Copy nodes into data_bags directory - when running in Vagrant mode,
    # chef-solo will look for nodes in data_bags/node/*, however in knife solo
    # mode nodes should be in nodes/*
    FileUtils.mkdir_p('data_bags/node')
    Dir['nodes/*'].each{ |f| FileUtils.cp(f,"data_bags/node") }

    chef.json = vagrant_json

    vagrant_json['run_list'].each do |item|
      chef.add_role(item) if item.start_with?("role[")
      chef.add_recipe(item) if item.start_with?("recipe[")
    end if vagrant_json['run_list']


    if vagrant_json['hostname']
      # Set chef node_name properly from node definition
      chef.node_name = "vagrant-#{vagrant_json['hostname']}"
    elsif vagrant_json['name']
      puts "WARNING: No node hostname found, using name instead!!!"
      chef.node_name = "vagrant-#{vagrant_json['name']}"
    else
      abort("Node has no name or hostname")
    end

  end
end


Vagrant.configure("2") do |config|


  config.vm.box =   "precise64-chef-client-omnibus-11.4.0-0.4"
  config.vm.box_url = "http://binary.aodn.org.au/static/boxes/precise64-chef-client-omnibus-11.4.0-0.4.box"

  # ssh options
  config.ssh.username = "vagrant"
  config.ssh.forward_agent = true

  # config.vm.network :forwarded_port, guest: port_pair[0].to_i, host: port_pair[1].to_i
	config.vm.network :private_network, ip: "10.11.12.13"

	node_name = 'julian1.org'

  node_file_path = "nodes"

  chef_log_level = ENV.fetch("CHEF_LOG", "info").downcase.to_sym

# puts "full path #{node_file_path}/#{node_name}.json"

  configure_virtualbox_provider config, node_name

   config.vm.define node_name do |node|
       configure_chef_solo_provisioning node, node_name, "#{node_file_path}/#{node_name}.json", chef_log_level
   end

end

