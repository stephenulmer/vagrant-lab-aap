# -*- mode: ruby -*-
# vi: set ft=ruby :

##
## These files must be copied into the ./bundle directory.
##
aap_bundle = "ansible-automation-platform-containerized-setup-bundle-2.6-8-aarch64.tar.gz"
aap_manifest = "aap-manifest.zip"

# Define the AAP server
aap_server = { name: "aap", admin_password: "redhat123" }

# Define an array of managed nodes
managed = [
  { name: "node1" },
  { name: "node2" },
  { name: "node3" }
]

Vagrant.configure("2") do |config|
  if Vagrant.has_plugin?('vagrant-registration')
    config.registration.username = ENV['RH_USERNAME']
    config.registration.password = ENV['RH_PASSWORD']
    config.registration.auto_attach = false  # if true breaks simple content access
  end

  config.vm.define aap_server[:name], primary: true do |aap|
    aap.vm.box = "slu/rhel-10.2-ansible"
    aap.vm.hostname = aap_server[:name]
    aap.vm.synced_folder ".", "/vagrant"

    aap.vm.provider "parallels" do |prl|
      prl.name = aap_server[:name]
      prl.update_guest_tools = false
      prl.memory = "16384"
      prl.cpus = 4
    end

    # aap.vm.provision "Generate SSH Key", type: "shell" do |p|
    #   p.privileged = false
    #   p.inline = "ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -N '' <<< 'n'"
    # end

    aap.vm.provision "Extract AAP Installer", type: "shell" do |p|
      p.privileged = false
      p.inline = <<-SHELL
        mkdir aap-installer
        tar -xzvf /vagrant/bundle/${AAP_BUNDLE} -C aap-installer --strip-components=1
        cp /vagrant/bundle/rh-manifest.zip aap-installer
        envsubst < /vagrant/bundle/inventory.in > aap-installer/inventory
      SHELL
      p.env = { "AAP_BUNDLE" => aap_bundle,
                "AAP_SERVER_FQDN" => aap_server[:name] + '.shared',
                "AAP_ADMIN_PASSWORD" => aap_server[:admin_password] }
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
  managed.each do |server|
    config.vm.define server[:name] do |node|
      node.vm.box = "slu/rhel-10.2"
      node.vm.hostname = server[:name]
      node.vm.synced_folder ".", "/vagrant", disabled: true
      
      node.vm.provider "parallels" do |prl|
        prl.name = server[:name]
        prl.update_guest_tools = false
        prl.memory = "4096"
        prl.cpus = 1
      end
    end
  end
end

