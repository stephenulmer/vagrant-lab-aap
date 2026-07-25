# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'

##
## Locate and load configuration data
##
config_path = File.join(File.dirname(__FILE__), 'lab-config.yml')
if File.exist?(config_path)
  puts "Loading lab-config.yml"
  lab = YAML.safe_load_file(config_path, permitted_classes: [Symbol])
else
  puts "Loading embedded YAML"
  yaml_data = <<~YAML
    server:
      name: aap

    managed:
      - name: managed-node
  YAML
  lab = YAML.safe_load(yaml_data, permitted_classes: [Symbol])
end


Vagrant.configure("2") do |config|
  if Vagrant.has_plugin?('vagrant-registration')
    config.registration.username = ENV['RH_USERNAME']
    config.registration.password = ENV['RH_PASSWORD']
    config.registration.auto_attach = false  # if true breaks simple content access
    config.registration.skip =  (ENV['RH_PASSWORD'].to_s.empty? &&
                                 ENV['RH_PASSWORD'].to_s.empty?) ? true : false
  end

  lab_server_name = lab.dig('server', 'name') || "aap"
  lab_server_box = lab.dig('server', 'box') || "slu/rhel-10.2-ansible"
  lab_server_bundle = lab.dig('server', 'bundle') || "ansible-automation-platform-containerized-setup-bundle-2.6-8-aarch64.tar.gz"
  lab_server_manifest = lab.dig('server', 'manifest') || "rh-manifest.zip"
  lab_server_admin_password = lab.dig('server', :admin_password) || "redhat123"

  config.vm.define lab_server_name do |aap|
    aap.vm.box = lab_server_box
    aap.vm.hostname = lab_server_name
    aap.vm.synced_folder ".", "/vagrant"

    aap.vm.provider "parallels" do |prl|
      prl.name = lab_server_name
      prl.update_guest_tools = false
      prl.memory = "16384"
      prl.cpus = 4
    end

    aap.vm.provision "Extract AAP Installer", type: "shell" do |p|
      p.privileged = false
      p.inline = <<-SHELL
        mkdir aap-installer
        tar -xzvf /vagrant/bundle/${AAP_BUNDLE} -C aap-installer --strip-components=1
        cp /vagrant/bundle/${AAP_MANIFEST} aap-installer
        envsubst < /vagrant/bundle/inventory.in > aap-installer/inventory
      SHELL
      p.env = { "AAP_BUNDLE" => lab_server_bundle,
                "AAP_MANIFEST" => lab_server_manifest,
                "AAP_SERVER_FQDN" => lab_server_name + '.shared',
                "AAP_ADMIN_PASSWORD" => lab_server_admin_password }
    end

    aap.vm.provision "Install AAP", type: "shell" do |p|
      p.privileged = false
      p.inline = <<-SHELL
        cd aap-installer
        ansible-playbook -i inventory ansible.containerized_installer.install
        ansible-playbook -i inventory /vagrant/bundle/lab_machine_credential.yml
        ansible-playbook -i inventory /vagrant/bundle/lab_inventory.yml
      SHELL
    end
  end

  # Generate managed VMs from data structure
  lab_managed_vms = lab.dig('managed') || [ { "name" => "managed-node" } ]

  lab_managed_vms.each do |vm|
    managed_node_name = vm.dig('name') || "nodeX"
    managed_node_box = vm.dig('box') || "slu/rhel-10.2"
    managed_node_communicator = vm.dig('communicator') || :ssh
    managed_node_boot_timeout = vm.dig('boot_timeout') || 300

    config.vm.define managed_node_name do |node|
      node.vm.box = managed_node_box
      node.vm.hostname = managed_node_name
      node.vm.communicator = managed_node_communicator
      node.vm.boot_timeout = managed_node_boot_timeout
      node.vm.synced_folder ".", "/vagrant", disabled: true
      node.registration.skip = true
      
      node.vm.provider "parallels" do |prl|
        prl.name = managed_node_name
        prl.update_guest_tools = false
        prl.memory = "4096"
        prl.cpus = 1
      end
    end
  end
end

