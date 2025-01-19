# Talos single node cluster

This is a short guide on how to set up a Talos single node cluster.  
It is based on the [official talos documentation](https://www.talos.dev/v1.9/talos-guides/install/virtualized-platforms/hyper-v/#pushing-config-to-the-nodes).  
Clone the repo and and download tools to start the setup.

```powershell
git clone https://github.com/bihalu/talos.git
cd talos
wget https://github.com/siderolabs/talos/releases/download/v1.9.2/metal-amd64.iso -OutFile metal-amd64.iso
wget https://github.com/siderolabs/talos/releases/download/v1.9.2/talosctl-windows-amd64.exe -OutFile talosctl.exe
wget https://dl.k8s.io/release/v1.32.0/bin/windows/amd64/kubectl.exe -OutFile kubectl.exe
wget https://github.com/derailed/k9s/releases/download/v0.32.7/k9s_Windows_amd64.zip -OutFile k9s_Windows_amd64.zip
Expand-Archive k9s_Windows_amd64.zip -DestinationPath tmp
copy tmp/k9s.exe .
Remove-Item -Recurse -Force tmp
Remove-Item k9s_Windows_amd64.zip
```

## Setup VM

* launch hyper-v manager -> virtmgmt.msc
* new vm
* name talos
* generation 2
* memory 4GB (no dynamic)
* network default switch
* hard disk 40GB
* mount iso -> metal-amd64.iso
* settings firmware boot order -> 1. dvd, 2. disk, 3. network
* settings security disable secure boot
* connect and boot

> Take note of control plane ip-address, you have to set it later.

## Install talos

```powershell
# set control plane IP variable
$CONTROL_PLANE_IP='172.26.91.19'

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
```
> Bootstrap takes a few minutes. Wait until stage is running and ready.

```powershell
# Generate kubeconfig
talosctl kubeconfig .

# Copy kubeconfig
mkdir -p ~/.kube
cp kubeconfig ~/.kube/config
```


Finaly you can access your cluster with k9s tool.

```powershell
k9s
```

![k9s](./k9s.png)

## Finish

> Shutdown the VM and unmount iso, so that the next time you don't boot from the iso.  
  In addition, it makes sense to create a snapshot of the VM.

Now have fun with your single node talos cluster ;-)
