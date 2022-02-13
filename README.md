# Kubernetes cluster using vagrant libvirtd as base


## Setup virtual machines

Start the vagrant machines:

```bash
vagrant up
```

Deploy the ansible playbook to install kubernetes binaries:

```bash
ansible-playbook -i inventory main.yml
```

## Deploy kubernetes

Configure kubernetes master node:

Connect to master node: example `vagrant ssh master`

```bash
sudo kubeadm init --pod-network-cidr=172.16.0.0/16 ## copy the output of this command to setup the nodes later
mkdir ~/.kube
cp /etc/kubernetes/admin ~/.kube/config
cp /etc/kubernetes/admin /vagrant
kubectl create -f https://projectcalico.docs.tigera.io/manifests/tigera-operator.yaml
kubectl create -f /vagrant/k8s-manifests/colico/custom-resources.yaml
```

Configure kubernetes worker nodes:

Connect to all three nodes: example `vagrant ssh node1`

```bash
sudo kubeadm join 192.168.121.56:6443 --token y6i3sk.g1cfocfndxr882bb ## this command is an example, use the output of the master node 
```

## Configure your workstation

On your workstation

```bash
cp shared/admin ~/.kube/config
kubectl get nodes ## This should return all 4 pods (1 master and 3 nodes)
```

## Setup ingest controller: nginx

Source: https://github.com/geerlingguy/kubernetes-101/tree/master/episode-06

First we need a loadbalancer to route traffic

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.11.0/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.11.0/manifests/metallb.yaml
kubectl apply -f shared/k8s-manifests/metallb/config-map.yaml
```

Now we install the nginx ingress controller

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install ingress-nginx ingress-nginx/ingress-nginx

kubectl get svc ingress-nginx-controller ## Get the external ip
```

## Setup Prometheus and Grafana

Source: https://github.com/geerlingguy/kubernetes-101/tree/master/episode-10

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

kubectl create namespace monitoring
helm install prometheus --namespace monitoring prometheus-community/kube-prometheus-stack

kubectl get deployments -n monitoring -w
```

After the deployment is finished, add a ingress controller configuration to Grafana

```bash
kubectl apply -f shared/k8s-manifests/grafana/grafana-ingress.yaml
```

Add a DNS A type record to your system pointing to the ingress controller IP address.

Example using /etc/hosts:
```config
10.0.0.30   grafana.k8s.test
```

## Setup NFS storage class

Source: https://github.com/geerlingguy/kubernetes-101/tree/master/episode-06 (outdated)

If you have a NFS storage, you can connect your cluster to it.

NFS storage class will allow you to use PVCs with ReadWriteMany mode.

```bash
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
    --set nfs.server=x.x.x.x \
    --set nfs.path=/exported/path
```


## References:

* [Kubernetes Setup Using Ansible and Vagrant](https://kubernetes.io/blog/2019/03/15/kubernetes-setup-using-ansible-and-vagrant/)
* [Jeff Geerling Kubernetes 101 Series](https://github.com/geerlingguy/kubernetes-101)
* [Calico Quickstart Guide](https://projectcalico.docs.tigera.io/getting-started/kubernetes/quickstart)
* [MetalLB Installation](https://metallb.universe.tf/installation/)
* [nfs-subdir-external-provisioner Github](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner)