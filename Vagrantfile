# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.define :master do |master|
    master.vm.box = "centos63"
    master.vm.box_url = "https://dl.dropbox.com/u/7225008/Vagrant/CentOS-6.3-x86_64-minimal.box"
    master.vm.network :private_network, ip: "192.168.50.4"
  end
  config.vm.define :agent do |agent|
    agent.vm.box = "centos63"
    agent.vm.box_url = "https://dl.dropbox.com/u/7225008/Vagrant/CentOS-6.3-x86_64-minimal.box"
  end

  # config.vm.network :forwarded_port, guest: 80, host: 8080

end
