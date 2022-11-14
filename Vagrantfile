IMAGE_NAME = "fedora-coreos"
WORKER_NODES = 2
LINUX_USER = "vagrant"

Vagrant.configure("2") do |config|
  config.ssh.insert_key = false
  config.vm.synced_folder ".", "/vagrant", disabled: true
  if LINUX_USER == "core"
    config.ssh.username = "core"
    config.ssh.private_key_path = "./core"
  end
  config.vm.provider :libvirt do |libvirt|
    libvirt.management_network_domain = "k8s.local"
    libvirt.management_network_name = "kubernetes"
    libvirt.management_network_mode = "nat"
    libvirt.management_network_address = "192.168.56.0/24"
  end

  config.vm.define "k8s-controlplane", primary: true do |controlplane|
    controlplane.vm.box = IMAGE_NAME
    controlplane.vm.hostname = "controlplane.k8s.local"
    controlplane.vm.provider :libvirt do |libvirt|
      libvirt.driver = "kvm"
      libvirt.cpus = 2
      libvirt.memory = 4096
    end
    controlplane.vm.provision "ansible" do |ansible|
      ansible.playbook = "ansible/k8s-controlplane.yml"
      ansible.extra_vars = {
        kubernetes_release: "1.25.4",
        ansible_user: LINUX_USER
      }
    end
  end

  (1..WORKER_NODES).each do |i|
    config.vm.define "k8s-node-#{i}" do |node|
        node.vm.box = IMAGE_NAME
        node.vm.hostname = "node-#{i}.k8s.local"
        node.vm.provider :libvirt do |libvirt|
          libvirt.driver = "kvm"
          libvirt.cpus = 2
          libvirt.memory = 2048
        end
        node.vm.provision "ansible" do |ansible|
            ansible.playbook = "ansible/k8s-workers.yml"
            ansible.extra_vars = {
              kubernetes_release: "1.25.4",
              ansible_user: LINUX_USER
            }
        end
        if i == WORKER_NODES
            node.vm.provision :ansible do |ansible|
                ansible.limit = "k8s-controlplane"
                ansible.playbook = "ansible/k8s-addons.yml"
                ansible.extra_vars = {
                  kubernetes_release: "1.25.4",
                  ansible_user: LINUX_USER
                }
            end
        end
    end
  end
end
