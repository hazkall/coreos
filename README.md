# CoreOS

## Install CoreOs on libvirt

```bash

## Create ignite file

docker run -i --rm quay.io/coreos/butane:release --pretty --strict < fcos.bu.yml > config.ign

sudo cp config.ign /var/lib/libvirt/images

sudo cp fedora-coreos-36.20221014.3.1-qemu.x86_64.qcow2 /var/lib/libvirt/images/ 

virt-install --connect="qemu:///system" --name="coreos" --vcpus="2" --memory="4096" --os-variant="fedora-coreos-stable" --os-type=linux --import --graphics=none --disk="size=10,backing_store=/var/lib/libvirt/images/fedora-coreos-36.20221014.3.1-qemu.x86_64.qcow2" --network bridge=virbr0 --qemu-commandline="-fw_cfg name=opt/com.coreos/config,file=/var/lib/libvirt/images/config.ign"

## Inside Coreos ##

ssh -i <private_key> core core@<ip-addres-libvirt>

sudo rpm-ostree install python3 crio kubelet-1.25.4 kubectl-1.25.4 kubeadm-1.25.4

```

## Create CoreOS Box with libvirt

```bash

./create_box.sh /var/lib/libvirt/images/coreos.qcow2

vagrant box add coreos.box --name fedora-coreos

```

## Cluster Bootstrap

```bash

vagrant up --provider=libvirt --no-parallel

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