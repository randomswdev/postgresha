postgres_nodes = 3
consul_password = "consulpassword"

Vagrant.configure(2) do |config|
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
    config.vm.box = "centos/8"
    config.vm.define "postgresha-#{i}" do |v|
      v.vm.hostname = "postgresha-#{i}.test.com"
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
    v.vm.box = "centos/7"
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
         "postgres" => ["postgresha-[1:#{postgres_nodes}]"],
         "consul_instances" => ["postgresha-services"],
         "consul_instances:vars" => {
            "consul_version" => "1.9.1",
            "consul_node_role" => "server",
            "consul_bootstrap_expect" => "true",
            "consul_acl_enable" => "true",
            "consul_acl_default_policy" => "deny",
            "consul_acl_master_token" => "#{consul_password}",
            "consul_iface" => "eth1",
            "consul_client_address" => "{{ ansible_facts[consul_iface].ipv4.address }}",
          }
      }
      ansible.galaxy_role_file = "provisioning/roles/requirements.yml"
      ansible.galaxy_roles_path = "provisioning/roles"
    end    

  end
end