# -*- mode: ruby -*-
# vi: set ft=ruby :

NODE_COUNT    = 3
BASE_IP       = "33.33.33"
IP_INCREMENT  = 10

seeds = []

(1..NODE_COUNT).each do |index|
  last_octet = index * IP_INCREMENT
  node_ip = "#{BASE_IP}.#{last_octet}"
  seeds << {'index' => index,
            'name' => "node#{index}",
            'ip' => node_ip}
end

Vagrant.configure("2") do |cluster|
  # Install latest chef on the client node, requires vagrant-omnibus plugin
  cluster.omnibus.chef_version = :latest

  # Configure caching, so that cache can be shared among nodes, minimising downloads. Requires vagrant-cachier plugin
  # Uncomment next line to enable cachier, seems to cause problems on windows
    cluster.cache.auto_detect = true

  # Enable berkshelf because it makes manages cookbooks much simpler. Required vagrant-berkshelf plugin
  cluster.berkshelf.enabled = true


  seeds.each do |seed|
    cluster.vm.define seed['name'] do |config|
      config.vm.box = "precise64"
      config.vm.box_url = "http://files.vagrantup.com/precise64.box"
      config.vm.provider(:virtualbox) { |v| v.customize ["modifyvm", :id, "--memory", 1024] }

      config.vm.hostname = seed['name']
      config.vm.network :private_network, ip: seed['ip']

      # Provision using Chef.
      config.vm.provision :chef_solo do |chef|
        chef.json = {
          :mono => {
            :install_method => "ppa"
          },
          :eventstore => {
            :version => "2.5.0rc4",
            :source_uri => "http://ha.geteventstore.com/showcase/",
            :bin_filename => "EventStore-Mono-v2.5.0rc4.tar.gz",
            :command => "./clusternode",
            :install_dir => "/usr/local/eventstore/",
            :executable_dir => "/usr/local/eventstore/EventStore-Mono-v2.5.0rc4",
            :config => {
              :internalIp => seed['ip'],
              :externalIp => seed['ip'],
              :internalHttpPort => 2113,
              :externalHttpPort => 2114,
              :internalTcpPort => 1111,
              :externalTcpPort => 1112,
              :clusterSize => NODE_COUNT,
              :useDnsDiscovery => false,
              :gossipSeed => seeds.reject{|s| s['index'] == seed['index']}.map{|i| "#{i['ip']}:2113"}
            }
          }
        }
        chef.add_recipe "eventstore"
      end
    end
  end
end
