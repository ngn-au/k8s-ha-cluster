# K8S CLUSTER
Kubernetes Multi Master Cluster

+ kubectl/v1.25.3
+ calico/cni:v3.24.5
+ kubernetesui/dashboard:v2.7.0
+ kubernetesui/metrics-scraper:v1.0.8
+ open-iscsi
+ helm

# PREREQUISITES

You need to have your ssh id already provisioned; otherwise run: ssh-keygen -t rsa 

Make sure you have copied your ssh key from ansible to the hosts prior to running playbooks, in our case - the vm template built in playbook 0- had both ssh installed and the key in authorized_keys

Or run the ansible-playbook command with -k to be prompted for password


# DOCS

k8s-ha:
  - 10.20.40.20 LOAD BALANCED (HA PROXY) VIRTUAL IP (KEEPALIVED)
  - 10.20.40.21 k8s-ha1 12vCPU | 16GB RAM | 24GB + 100GB DISK (Cluster Master + Worker)
  - 10.20.40.22 k8s-ha2 12vCPU | 16GB RAM | 24GB + 100GB DISK (Cluster Master + Worker)
  - 10.20.40.23 k8s-ha3 12vCPU | 16GB RAM | 24GB + 100GB DISK (Cluster Master + Worker)
  - 10.20.40.30 k8s-halb1 4vCPU | 4GB RAM | 24GB (External Round Robbin Load Balancer)
  - 10.20.40.29 k8s-halb2 4vCPU | 4GB RAM | 24GB


# INSTALL

Edit to your needs each file found:

`grep 10.20.40 * -R | awk '{print $1}' | sort |uniq`

***Make sure you change everything exactly. Best to use find / replace***

***Note:*** 

This playbook is designed to run with a template of Debian 11 fresh from netinst, which is automatically provisioned in playbook `0_create_vms.yaml` from a ready made Appliance NOT provided. If you want to ansible to provision your VMs a template must be created in XenCenter/XCP-ng Center call it `"Debian 11"`.


Therefore, if you don't have XenServer / XCP-ng edit start-k8s-ha-cluster.yaml and add `"when: false"` to first task called `"Create vms using xe (XCP-ng / XenServer"`

AND 

Create your VM's with the default values `k8s-ha` or with your own edited version  


Then once your vms are up contactable and ssh-able as root from your ansible host

From inside ansible directory run: 

`
ansible-playbook -i hosts start-k8s-ha-cluster.yaml
`







# KUBECTL ACCESS 
ssh to 10.20.40.21-23 to run kubectl or helm - (helm repo do not sync between nodes)

- If you have kubectl locally
`scp 10.20.40.21:/etc/kubernetes/admin.conf ~/.kube/config`

# CONFIRM PODS 
`kubectl get pods -A`

# TO VIEW THE DASHBOARD

You need the NodePort from the Kubernetes Dashboard service, to find it: 

` kubectl describe service -nkubernetes-dashboard | grep -i nodeport |grep unset `

Open up your browser to any of the nodes

https://10.20.40.21:< NodePort >/

https://10.20.40.22:< NodePort >/

https://10.20.40.23:< NodePort >/
