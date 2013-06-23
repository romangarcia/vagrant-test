# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.define :master do |master|
    master.vm.provider "virtualbox" do |v|
      v.name = "master_vm"
    end
    master.vm.box = "centos63"
    master.vm.box_url = "https://dl.dropbox.com/u/7225008/Vagrant/CentOS-6.3-x86_64-minimal.box"
    master.vm.network :private_network, ip: "192.168.50.10"
    master.vm.network :forwarded_port, host: 3000, guest: 3000
  end

  config.vm.define :agent do |agent|
    agent.vm.provider "virtualbox" do |v|
      v.name = "agent_vm"
    end  
    agent.vm.box = "centos63"
    agent.vm.box_url = "https://dl.dropbox.com/u/7225008/Vagrant/CentOS-6.3-x86_64-minimal.box"
	  agent.vm.network :private_network, ip: "192.168.50.20"
  end

  config.vm.define :winagent do |winagent|
   # Configure base box parameters
    winagent.vm.provider "virtualbox" do |v|
      v.name = "winagent_vm"
      v.gui = true
    end   
    winagent.vm.guest = :windows  
    winagent.vm.box = "win2008"
    winagent.vm.box_url = "http://dl.dropbox.com/u/58604/vagrant/win2k8r2-core.box"    
    #winagent.vm.boot_mode = :gui

    # Max time to wait for the guest to shutdown
    winagent.windows.halt_timeout = 15

    # Admin user name and password
    winagent.winrm.username = "vagrant"
    winagent.winrm.password = "vagrant"

    # Port forward WinRM and RDP
    winagent.vm.network :private_network, ip: "192.168.50.30"
    winagent.vm.network :forwarded_port, guest: 3389, host: 3389
    winagent.vm.network :forwarded_port, guest: 5985, host: 5985
  end

end
