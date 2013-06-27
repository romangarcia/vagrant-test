# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.define :master do |master|
    master.vm.provider "virtualbox" do |v|
      v.name = "master_vm"
    end
    master.vm.box = "centos63"
    master.vm.box_url = "~/vagrant-boxes/centos-6-3-puppet-3-1-master.box"
    master.vm.network :private_network, ip: "192.168.50.10"
  end

  config.vm.define :agent do |agent|
    agent.vm.provider "virtualbox" do |v|
      v.name = "agent_vm"
    end  
    agent.vm.box = "centos63"
    agent.vm.box_url = "~/vagrant-boxes/centos-6-3-puppet-3-1-agent.box"
    agent.vm.network :private_network, ip: "192.168.50.20"
  end

  config.vm.define :winagent do |winagent|
   # Configure base box parameters
    winagent.vm.provider "virtualbox" do |v|
      v.name = "winagent_vm"
      v.gui = true
    end   
    winagent.vm.box = "win2008"
    winagent.vm.box_url = "~/vagrant-boxes/win-2008-puppet-3-1-agent.box"    
    #winagent.vm.boot_mode = :gui

    # Port forward WinRM and RDP
    winagent.vm.network :private_network, ip: "192.168.50.30"
  end

end
