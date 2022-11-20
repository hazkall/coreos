# CoreOS

## Requeriments

```bash

ssh key-gen

## Insert your public key on fcos.bu.yml

variant: fcos
version: 1.0.0
passwd:
 users:
 — name: core
 ssh_authorized_keys:
 — ssh-rsa AAAAB3…YOUR_PUBLIC_SSH_KEY_GOES_HER

```

## AppArmor Config

```bash

echo "  /var/lib/libvirt/ignition/* r," >> /etc/apparmor.d/abstractions/libvirt-qemu
echo "  /var/lib/libvirt/base/* r," >> /etc/apparmor.d/abstractions/libvirt-qemu
echo "  /var/lib/libvirt/* r," >> /etc/apparmor.d/abstractions/libvirt-qemu

sudo systemctl restart apparmor.service

```

## Install CoreOs on libvirt

```bash

# CoreOS ISO download -> https://getfedora.org/en/coreos/download?tab=metal_virtualized&stream=stable&arch=x86_64

xz -vd fedora-coreos-36.20221014.3.1-qemu.x86_64.qcow2.xz

## Create ignite file

docker run -i --rm quay.io/coreos/butane:release --pretty --strict < fcos.bu.yml > config.ign

sudo cp config.ign /var/lib/libvirt/ignition/

sudo mv fedora-coreos-36.20221014.3.1-qemu.x86_64.qcow2 /var/lib/libvirt/base/

virt-install --connect="qemu:///system" --name="coreos" --vcpus="2" --memory="4096" --os-variant="fedora-coreos-stable" --import --graphics=none --disk="size=10,backing_store=/var/lib/libvirt/base/fedora-coreos-36.20221030.3.0-qemu.x86_64.qcow2" --network bridge=virbr0 --qemu-commandline="-fw_cfg name=opt/com.coreos/config,file=/var/lib/libvirt/ignition/config.ign"

## Inside Coreos ##

ssh -i <private_key> core core@<ip-addres-libvirt>

sudo rpm-ostree install python3 crio kubelet-1.25.4 kubectl-1.25.4 kubeadm-1.25.4

sudo shutdown now

```

## Create CoreOS Box with libvirt

```bash

sudo ./create_box.sh /var/lib/libvirt/images/coreos.qcow2

vagrant box add coreos.box --name fedora-coreos --force

virsh undefine coreos --remove-all-storage

rm -f coreos.box

```

## Cluster Bootstrap

```bash

export VAGRANT_DEFAULT_PROVIDER=libvirt

vagrant up --no-parallel --provider=libvirt

vagrant ssh k8s-controlplane

kubectl get nodes -o wide

```

## Destroy cluster

```bash

vagrant destroy --force

```

## K8S Cluster

- CoreOS
- K8S 1.25.4
- Cilium
- CRIO
- Security Best Practics on cluster bootstrap
- Addons: Metric-Server
