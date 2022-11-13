IMAGE_NAME = "fedora-coreos"
NODES = 2

Vagrant.configure("2") do |config|
  config.ssh.insert_key = false
  # config.ssh.username = "coreos"
  # config.ssh.private_key_path = "./coreos"
  config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.define "k8s-controlplane", primary: true do |controlplane|
    controlplane.vm.box = IMAGE_NAME
    controlplane.vm.hostname = "k8s-controlplane"
    controlplane.vm.provider :libvirt do |libvirt|
      libvirt.driver = "kvm"
      libvirt.cpus = 2
      libvirt.memory = 4096
    end
    controlplane.vm.provision "ansible" do |ansible|
      ansible.playbook = "ansible/k8s-controlplane.yml"
      ansible.extra_vars = {
          kubernetes_release: "1.25.4",
          hostname: "k8s-controlplane",
      }
    end
  end

  (1..NODES).each do |i|
    config.vm.define "k8s-node-#{i}" do |node|
        node.vm.box = IMAGE_NAME
        node.vm.hostname = "k8s-node-#{i}"
        node.vm.provider :libvirt do |libvirt|
          libvirt.driver = "kvm"
          libvirt.cpus = 2
          libvirt.memory = 2048
        end
        node.vm.provision "ansible" do |ansible|
            ansible.playbook = "ansible/k8s-workers.yml"
            ansible.extra_vars = {
                kubernetes_release: "1.25.4",
                hostname: "k8s-node-#{i}",
            }
        end
        if i == NODES
            node.vm.provision :ansible do |ansible|
                ansible.limit = "k8s-controlplane"
                ansible.playbook = "ansible/k8s-addons.yml"

            end
        end
    end
  end
end
