# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "generic/ubuntu2004"

#---------- nfs ----------
  config.vm.define "nfsserver_192.168.133.11" do |server|
    server.vm.network "private_network", ip: "192.168.33.11"
    server.vm.network "private_network", ip: "192.168.133.11"
    server.vm.hostname = "nfs"
    server.vm.provider "libvirt" do |kvm|
      kvm.storage_pool_name = "data"
      kvm.memory = 4096
      kvm.cpus = 2
      kvm.machine_type = "q35"
      kvm.kvm_hidden = true
    end
    server.vm.provision "shell", privileged: false, inline: <<-SHELL
      sudo ip link set eth1 mtu 1450
      sudo apt-get update
      sudo apt-get install -y docker.io
      sudo docker pull itsthenetwork/nfs-server-alpine
      sudo mkdir -p /exports
      sudo docker run -itd --name nfs --rm --privileged -p 2049:2049 -p 111:111/udp -v /exports:/exports -e SHARED_DIRECTORY=/exports itsthenetwork/nfs-server-alpine:latest
    SHELL
  end
#--------------------
end
