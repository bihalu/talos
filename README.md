# Talos single node cluster

This is a short guide on how to set up a Talos single node cluster.  
It is based on the [official talos documentation](https://www.talos.dev/v1.9/talos-guides/install/virtualized-platforms/hyper-v/#pushing-config-to-the-nodes).  
All required tools like talosctl, kubectl or the talos ios image are included in this repo.  
Clone the repo and you can start the setup.

```powershell
git clone https://github.com/bihalu/talos.git
cd talos
```

## Setup VM

* launch hyper-v manager
* new vm
* name talos
* generation 2
* memory 4GB (no dynamic)
* network default switch
* hard disk 40GB
* mount iso -> metal-amd64-v1.9.2-dirty.iso
* settings security disable secure boot
* connect and boot

> Take note of control plane ip-address, you have to set it later.

## Install talos

```powershell
# set control plane IP variable
$CONTROL_PLANE_IP='172.26.92.74'

# Generate talos config
talosctl gen config talos-cluster https://$($CONTROL_PLANE_IP):6443 --output-dir .

# Apply config to control plane node
talosctl apply-config --insecure --nodes $CONTROL_PLANE_IP --file .\controlplane.yaml
```

> Now talos VM is installing and after a minute stage should be booting, kubelet healthy and connectivity OK.

![booting](./booting.png)

## Bootstrap cluster

```powershell
# Copy talosconfig
mkdir -p ~/.talos
cp talosconfig ~/.talos/config

# Use following command to set node and endpoint permanantly in config so you dont have to type it everytime
talosctl config endpoint $CONTROL_PLANE_IP
talosctl config node $CONTROL_PLANE_IP

# Bootstrap cluster
talosctl bootstrap

# Generate kubeconfig
talosctl kubeconfig .

# Copy kubeconfig
mkdir -p ~/.kube
cp kubeconfig ~/.kube/config
```

> Bootstrap takes a few minutes. Wait until stage is running and ready.

Finaly you can access your cluster with k9s tool.

```powershell
k9s
```

![k9s](./k9s.png)

## Finish

> Shutdown the VM and unmount iso, so that the next time you don't boot from the iso.  
  In addition, it makes sense to create a snapshot of the VM.

Now have fun with your single node talos cluster ;-)
