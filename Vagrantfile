$master_script = <<SCRIPT
#!/bin/bash

apt-get install curl -y
apt-get update
export DEBIAN_FRONTEND=noninteractive
apt-get -q -y --force-yes install dnsmasq
SCRIPT

$hosts_script = <<SCRIPT
cat > /etc/hosts <<EOF
127.0.0.1       localhost

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

EOF
SCRIPT

$slave_script = <<SCRIPT
cat > /etc/dhcp/dhclient.conf <<EOF
supersede domain-name-servers 10.211.55.100;
EOF
sudo dhclient
SCRIPT

numNodes = 3
ipAddrPrefix = "10.200.50.10"

Vagrant.configure("2") do |config|

  # Define base image
  config.vm.box = "ubuntu/trusty64"

  # Manage /etc/hosts on host and VMs
  config.hostmanager.enabled = false
  config.hostmanager.manage_host = true
  config.hostmanager.include_offline = true
  config.hostmanager.ignore_private_ip = false

  config.vm.define :master do |master|
    master.vm.provider :virtualbox do |v|
      v.name = "master"
    end
    master.vm.network :private_network, ip: "10.200.50.9"
    master.vm.hostname = "master"
    master.vm.provision :shell, :inline => $hosts_script
    master.vm.provision :hostmanager
    master.vm.provision :shell, :inline => $master_script
  end


	1.upto(numNodes) do |num|
		nodeName = ("node" + num.to_s).to_sym
		config.vm.define nodeName do |node|
			node.vm.box = "ubuntu/trusty64"
			node.vm.network :private_network, ip: ipAddrPrefix + num.to_s
            node.vm.provision :shell, :inline => $slave_script
			node.vm.provider "virtualbox" do |v|
				v.name = "Cluster Node " + num.to_s
			end
		end
	end

end