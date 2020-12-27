postgres_nodes = 3
service_iface = "eth1"
cluster_name = "portal-postgresql"
consul_password = "consulpassword"
patroni_replication_password = "replicationpassword"
patroni_superuser_password = "patronipassword"

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
      v.vm.network "private_network", ip: "192.168.56.9#{i}"
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
    v.vm.network "private_network", ip: "192.168.56.90"
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
         "services:vars" => {
            "service_iface" => "#{service_iface}"
         },
         "postgres" => ["postgresha-[1:#{postgres_nodes}]"],
         "postgres:vars" => {
            "service_iface" => "#{service_iface}",
            
            "patroni_scope" => "#{cluster_name}",

            "patroni_bootstrap_dcs_synchronous_mode" => "true",
            "patroni_bootstrap_dcs_synchronous_mode_strict" => "true",

            "patroni_postgresql_version" => "9.6",
            "patroni_postgresql_connect_address" => "{{ ansible_facts['#{service_iface}'].ipv4.address }}:5432",
            "patroni_replication_password" => "#{patroni_replication_password}",
            "patroni_superuser_password" => "#{patroni_superuser_password}",

            "patroni_restapi_connect_address" => "{{ ansible_facts['#{service_iface}'].ipv4.address }}:8008",

            "patroni_dcs" => "consul",
            "patroni_consul_host" => "{{ hostvars[groups['consul_instances'][0]]['ansible_#{service_iface}']['ipv4']['address'] }}:{{ patroni_consul_port | default(8500) }}",
            "patroni_consul_token" => "#{consul_password}",
            "patroni_consul_register_service" => "true",

            "postgresql_yum_repo_pkg_version" => "9.6.20-1PGDG.rhel8",
            # Fix the role and remove this
            "postgresql_yum_repo_pkg_name" => "pgdg-redhat-repo-latest.noarch.rpm"
         },
         "consul_instances" => ["postgresha-services"],
         "consul_instances:vars" => {
            "consul_version" => "1.9.1",
            "consul_node_role" => "server",
            "consul_bootstrap_expect" => "true",
            "consul_acl_enable" => "true",
            "consul_acl_default_policy" => "deny",
            "consul_acl_master_token" => "#{consul_password}",
            "consul_iface" => "#{service_iface}",
            "consul_client_address" => "{{ ansible_facts['#{service_iface}'].ipv4.address }}"
          }
      }
      ansible.galaxy_role_file = "provisioning/roles/requirements.yml"
      ansible.galaxy_roles_path = "provisioning/roles"
    end    

  end
end

# https://www.cybertec-postgresql.com/en/patroni-setting-up-a-highly-available-postgresql-cluster/
# https://www.opsdash.com/blog/postgres-getting-started-patroni.html
# https://www.alibabacloud.com/blog/how-to-set-up-a-highly-available-postgresql-cluster-using-patroni-on-ubuntu-16-04_594477