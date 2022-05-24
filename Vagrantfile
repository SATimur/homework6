# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
  config.vm.box_version = "2004.01"

 config.vm.provider "virtualbox" do |v|
    v.memory = 256
    v.cpus = 1
 end
 config.vm.define "nginx" do |nginx|
  nginx.vm.network "private_network", ip: "192.168.56.10",
  virtualbox_intnet: "net1"
    nginx.vm.hostname = "nginx"
 end
  
   config.vm.provision "shell", inline: <<-SHELL
    yum install -y \
      redhat-lsb-core \
      wget \
      rpmdevtools \
      rpm-build \
      createrepo \
      yum-utils \
      yum update
  #   apt-get update
  #   apt-get install -y apache2
  SHELL
end