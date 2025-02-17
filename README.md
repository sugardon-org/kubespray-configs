# kubespray-configs

## Pre-requisites

1. kubespray
   ```bash
   git clone https://github.com/kubernetes-sigs/kubespray.git
   ```
## sugardon01

1. kubespray
   ```bash
   cd kubespray
   git checkout v2.24.1
   ```
1. run
   ```bash
   cp ../sugardon01/hosts.ini inventory/sample/
   ansible-playbook -i inventory/sample/hosts.ini cluster.yml -e "@../sugardon01/values.yaml" --skip-tags "kube-proxy" --become -vvv
   cp inventory/sample/artifacts/admin.conf ../sugardon01
   cd ../
   ```
1. kubectl
   ```bash
   kubectl get nodes --kubeconfig=sugardon01/admin.conf
   ```

## Troubleshooting

### cilium not work in Ubuntu 22.04

- https://github.com/cilium/cilium/issues/20125
- https://github.com/kubernetes-sigs/kubespray/issues/9039
- cilium upgrade
  - https://github.com/kubernetes-sigs/kubespray/pull/9225

Solution

```bash
# ssh to node
ssh -i .ssh/id_rsa sugardon_admin@${IP}
sudo -i

# problem
sysctl -a | grep '\.rp_filter'
#net.ipv4.conf.all.rp_filter = 0
#net.ipv4.conf.cilium_host.rp_filter = 2
#net.ipv4.conf.cilium_net.rp_filter = 2
#net.ipv4.conf.cilium_vxlan.rp_filter = 2
#net.ipv4.conf.default.rp_filter = 2
#net.ipv4.conf.ens3.rp_filter = 2
#net.ipv4.conf.ens4.rp_filter = 2
#net.ipv4.conf.ens5.rp_filter = 2
#net.ipv4.conf.kube-ipvs0.rp_filter = 2
#net.ipv4.conf.lo.rp_filter = 2
#net.ipv4.conf.lxc4918bfba8cf1.rp_filter = 2
#net.ipv4.conf.lxc_health.rp_filter = 0
#net.ipv4.conf.nodelocaldns.rp_filter = 2

# run
sed -i -e '/net.ipv4.conf.*.rp_filter/d' $(grep -ril '\.rp_filter' /etc/sysctl.d/ /usr/lib/sysctl.d/)
sysctl -a | grep '\.rp_filter' | awk '{print $1" = 0"}' > /etc/sysctl.d/1000-cilium.conf

# reload
sysctl --system

# confirm
sysctl -a | grep '\.rp_filter'
kubectl get pods --all-namespaces
```
