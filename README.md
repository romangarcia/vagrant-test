OJO!!!
- Acordate de correr todo con SUDO!
- Fijate que las VM puedan verse entre si (VagrantFile, hosts, etc)

Preparar las VMs
=========================

1)Instalar Vagrant

2) 
# vagrant init

3) Modificar el VagrantFile
- Levantar un entorno de N VMs con private-networking
- Elegir un box - Yo elegi CentOS 6.3 minimal desde http://www.vagrantbox.es
- Usar IP estatica para el Master
- Forwardear el puerto del Puppet-Dashboard para accederlo desde la PC host

Ejemplo del script:
-------------------
# SCRIPT VAGRANT PARA PUPPET ----------------------------
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.define :master do |master|
    master.vm.box = "centos63"
    master.vm.box_url = "https://dl.dropbox.com/u/7225008/Vagrant/CentOS-6.3-x86_64-minimal.box"
    master.vm.network :private_network, ip: "192.168.50.4"
	master.vm.network :forwarded_port, host: 3000, guest: 3000
  end
  config.vm.define :agent do |agent|
    agent.vm.box = "centos63"
    agent.vm.box_url = "https://dl.dropbox.com/u/7225008/Vagrant/CentOS-6.3-x86_64-minimal.box"
  end

  # config.vm.network :forwarded_port, guest: 80, host: 8080

end
# SCRIPT VAGRANT PARA PUPPET ----------------------------

4) 
# vagrant up

Instalar Puppet (MASTER): 
=========================
(http://docs.puppetlabs.com/guides/installation.html)

1)
# vagrant ssh master

2) Agregar repositorio de Puppet
sudo rpm -ivh http://yum.puppetlabs.com/el/6/products/i386/puppetlabs-release-6-7.noarch.rpm

3) Instalar Master
sudo yum install puppet-server (en el master)

Instalar Puppet (AGENT): 
=========================
(http://docs.puppetlabs.com/guides/installation.html)

1)
# vagrant ssh agent

2) Agregar repositorio de Puppet
sudo rpm -ivh http://yum.puppetlabs.com/el/6/products/i386/puppetlabs-release-6-7.noarch.rpm

3) Instalar Agente
sudo yum install puppet

Configurar hosts y firmar certificados:
=========================

1) Cambiar HOSTNAME (en ambos)
# sudo vi /etc/sysconfig/network
> HOSTNAME=agent.localdomain (master.localdomain en el master)

# sudo vi /etc/hosts
> 192.168.100.100  master.localdomain
> 192.168.100.200  agent.localdomain

2) Configurar Puppet (Agent)
# sudo vi /etc/puppet/puppet.conf
> [agent]
> report = true
> pluginsync = true
> certname = agent.localdomain
> server = master.localdomain

3) Firmar certificados

a) En el agent:
# sudo puppet agent --test --debug
>> deberian aparecer las siguientes lineas:
Info: Creating a new SSL certificate request for agent.localdomain
Exiting; no certificate found and waitforcert is disabled

b) En el master, despues de (A):
# sudo puppet cert list --all
>> deberia listarse un "certificate_request" para "agent.localdomain"

# sudo puppet cert --sign agent.localdomain
>> deberia aparecer el "sign" del certificado, y el remove del "certificate_request"

----------------------------
!!! En caso de problemas !!!
----------------------------
 a) Usar --debug en el AGENT y --no-daemonize --verbose --debug en el MASTER
 b) Eliminar los certificados generados y los requests (ojo si tenias cert aprobados los vas a perder!)
   - AGENT: sudo rm -rf /var/lib/puppet/ssl
   - MASTER: sudo puppet cert clean --all
 c) Fijate si el firewall te deja llegar al master puerto 8140 (ver iptables)
 d) Asegurate que /etc/hosts este bien
 e) Asegurate que el comando: "hostname" te devuelve el nombre que esperabas (quiza te falto reiniciar la VM)
 
Instalar el Dashboard:
======================
Instalar MySQL
# sudo yum install mysql-server.x86_64

Crear DB
# sudo service mysqld start
# sudo mysql -u root
CREATE DATABASE dashboard CHARACTER SET utf8;
CREATE USER 'dashboard'@'localhost' IDENTIFIED BY 'dashboard';
GRANT ALL PRIVILEGES ON dashboard.* TO 'dashboard'@'localhost';

Instalar el Dashboard
# sudo yum install puppet-dashboard

Configurarlo ( user/password y nombre de base )
# sudo vi /usr/share/puppet-dashboard/config/database.yml

Arrancar el server
# sudo service puppet-dashboard start

- Configurar MySQL para auto-arranque
# sudo /sbin/chkconfig --add mysqld
# sudo /sbin/chkconfig mysqld on

- Configurar para reportar al dashboard
# sudo vi /etc/puppet/puppet.conf (on each agent)
  [agent]
    report = true
# sudo vi /etc/puppet/puppet.conf (on puppet master)
  [master]
    reports = store, http
    reporturl = http://dashboard.example.com:3000/reports/upload	
	
- Configurar para usar el Dashboard como ENC (External Node Classifier) --- OPCIONAL
  [master]
    node_terminus = exec
    external_nodes = /usr/bin/env PUPPET_DASHBOARD_URL=http://localhost:3000 /opt/puppet-dashboard/bin/external_node	
	
- Configurar el Dashboard y los Workers para auto-arranque
# sudo chkconfig puppet-dashboard on
# sudo chkconfig puppet-dashboard-workers on

Agregar al final
>> env RAILS_ENV=production /usr/share/puppet-dashboard/script/delayed_job -p dashboard -n 4 -m start	

Reiniciar el Master
# sudo service puppetmaster restart

Instalar una VM Windows 2008
============================

- Modificar VagrantFile
  config.vm.define :winagent do |winagent|
    winagent.vm.box = "win2008"
    winagent.vm.box_url = "http://dl.dropbox.com/u/58604/vagrant/win2k8r2-core.box"
    winagent.vm.guest = :windows
    winagent.windows.halt_timeout = 15
    winagent.winrm.username = "vagrant"
    winagent.winrm.password = "vagrant"    
    winagent.vm.network :private_network, ip: "192.168.50.6"
    winagent.vm.network :forwarded_port, guest: 5985, host: 5985
  end
  
>>> Iniciar la VM con Virtualbox y asegurarse lo siguiente
-Enable WinRM
   winrm quickconfig -q
   winrm set winrm/config/winrs @{MaxMemoryPerShellMB="512"}
   winrm set winrm/config @{MaxTimeoutms="1800000"}
   winrm set winrm/config/service @{AllowUnencrypted="true"}
   winrm set winrm/config/service/auth @{Basic="true"}
Create a vagrant user, for things to work out of the box username and password should both be "vagrant".
Turn off UAC (Msconfig)
Disable complex passwords  
<<<<
  
- Instalar el plugin vagrant-windows
# vagrant plugin install vagrant-windows

