# Ansible playbooks to setup cross-datacenters Kubernetes HA (multi-master)
This repo provides Ansible playbooks to setup Kubernetes HA cluster. The playbooks are mainly inspired by Kubeadm documentation and other ansible tentatives on github. The playbooks could be used separately or as one playbook for a fully fledged HA cluster.

# Prerequisites:
- Servers running CentOS 7.x or Ubuntu 18.04
- pip:
    ```
    wget http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
    rpm -ivh epel-release-latest-7.noarch.rpm
    yum install -y python-pip
    ```

    or

    ```
    apt install -y python-pip
    ```
- ansible:
    ```pip install ansible```

- Setup ssh access from master to workers:
    ```ssh-copy-id -i ~/.ssh/id_rsa.pub <user@host>```

# Environment preparation:
- Clone this repo in the machine that you want to use as ansible manager (can be your laptop or any other machine that has ssh access to the target machines):
 ```
 git clone git@github.com:tmagalhaes1985/ansible-kubernetes-ha-cluster.git
 cd kl-ansible-k8s-cluster
 ```

- Define your machines in inventory file such as:
```myhostname ansible_usehost=<ip> ansible_user=<user>```

There are different groups being defined and used in the ```inventory``` file:
```
[aws-k8s-masters] # these are all masters of AWS
[gcp-k8s-workers] # these are all workers of Google Cloud Platform
[dgo-k8s-workers] # these are all worker nodes of DigitalOcean
[vmw-k8s-workers] # these are all worker nodes of VMware vSphere
```
We can have as many data centers as we need. For each data center, define the masters and workers and add them to `[k8s-masters:children]`, `[k8s-workers:children]`, and `[k8s-nodes:children]`. 

You can check that you can ping all the machines:

```ansible -m ping all -i inventory```

# Install a highly available kubernetes using kubeadm
You can now run ```config-all.yaml``` playbook to get your cluster setup.

You can also run the different playbooks separately for different purposes (setting up docker, etcd, keepalived, kubeadm etc).

```ansible-playbook -i inventory playbooks/config-all.yaml```

# config-all.yaml includes
- Add the required repos
- Install ntpd
- Install docker
- Install kubeadm, kubelet and kubectl
- Configure firewall ports
- Generate etcd certificates and install HA etcd cluster on all the master nodes
- Install HAProxy
- Install Keepalived and setup vip (optional, use only when you have a vip)
- Setup kubernetes masters
- Add worker nodes to the cluster
- Reconfigure nodes and components to communicate through HAProxy
- Run a smoke test

# Cleaning up the install
If you need to restart the process using kubeadm reset, please use the ```cleanup-all.yaml``` playbook that deletes the state from all machines in your inventory. Some of the commands might fail but you can ignore that.