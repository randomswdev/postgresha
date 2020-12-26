postgres_nodes = 3

Vagrant.configure(2) do |config|
  # Specifying the box we wish to use
  config.vm.box = "centos/8"

  # vagrant plugin install vagrant-hostmanager
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = false
  config.hostmanager.manage_guest = true
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = true
  
  # vagrant plugin install vagrant-proxyconf
  if Vagrant.has_plugin?("vagrant-proxyconf")
    config.proxy.http     = "http://10.0.2.2:3128/"
    config.proxy.https    = "http://10.0.2.2:3128/"
    config.proxy.no_proxy = "localhost,127.0.0.1,10.0.2.15,10.0.0.0/8,.test.com,192.168.56.0/24"
  end  
  
  # Iterating the nodes loop
  (1..postgres_nodes).each do |i|
    # Defining VM properties
    config.vm.define "postgresha#{i}" do |v|
      v.vm.hostname = "postgresha#{i}.test.com"
      v.vm.network "private_network", ip: "192.168.56.10#{i}"
      v.vm.provider "virtualbox" do |vb|
        vb.name = "postgresha-#{i}"
        vb.memory = 4096
        vb.cpus = 2
      end
    end
  end

  # Build the services machine
  config.vm.define "postgresha-services" do |v|
    v.vm.hostname = "postgresha-services.test.com"
    v.vm.network "private_network", ip: "192.168.56.100"
    v.vm.provider "virtualbox" do |vb|
      vb.name = "postgresha-services"
      vb.memory = 4096
      vb.cpus = 2
    end

    v.vm.provision "ansible" do |ansible|
      ansible.limit = "all"
      ansible.playbook = "provisioning/playbook.yml"
      ansible.groups = {
         "services" => ["postgresha-services"],
         "postgres" => ["postgresha[1:#{postgres_nodes}]"],
    }
    end    

  end
end