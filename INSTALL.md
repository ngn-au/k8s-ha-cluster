1. Prepare 5 debian 11 machines with disk storage attached
  - 10.20.40.21 k8s-ha1 12vCPU | 16GB RAM | 24GB + 100GB DISK (Cluster Master + Worker)
  - 10.20.40.22 k8s-ha2 12vCPU | 16GB RAM | 24GB + 100GB DISK (Cluster Master + Worker)
  - 10.20.40.23 k8s-ha3 12vCPU | 16GB RAM | 24GB + 100GB DISK (Cluster Master + Worker)
  - 10.20.40.30 k8s-halb1 4vCPU | 4GB RAM | 24GB (External Round Robbin Load Balancer)
  - 10.20.40.29 k8s-halb2 4vCPU | 4GB RAM | 24GB

# RUN ANSIBLE PLAYBOOKS
- 0_create_vms.yaml
- 1_kube_user.yaml
- 2_install_k8s.yaml
- 3_setup_load_balancers.yaml

# MANUAL STEPS 

# On first k8s-ha1 
kubeadm init --control-plane-endpoint="10.20.40.20:6443" --upload-certs --apiserver-advertise-address=10.20.40.21

# get the key
kubeadm init phase upload-certs --upload-certs
kubeadm token create --print-join-command --certificate-key e3a58787105ee2d7ce34d1291d9b8b8d466ae65bed64c9374c96556d67914a01


# Install calico 
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml 


# On remaining and subsequent masters 
kubeadm join 10.20.40.20:6443 --token zwmx5z.b96tfjdnjfh7d2g2 --discovery-token-ca-cert-hash sha256:2087d77559b5de647e2ab9326a425a6c0843d4716500421e7d3c272b39691507 --control-plane --certificate-key d99853cfdb74a7128bbd4e9fd69df0cd0e3c4ec977930218292530ac67a33c2b --apiserver-advertise-address=10.20.40.24


# Untaint the cluster nodes so that pods can run on them
kubectl taint nodes k8s-ha1 node-role.kubernetes.io/control-plane-
kubectl taint nodes k8s-ha2 node-role.kubernetes.io/control-plane-
kubectl taint nodes k8s-ha3 node-role.kubernetes.io/control-plane-


# install dashboard 
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml


helm install openebs-cstor openebs-cstor/cstor -n openebs --create-namespace

kubectl apply -f https://openebs.github.io/charts/cstor-operator.yaml



# create admin user 
kubectl apply -f create-admin-user.yaml 

# CREATE CSTORE STORAGE POOL AND CLASS
kubectl get bd -n openebs
check cspc.yaml and update node names / blockdevices 

# Create the Pool
kubectl apply -f cspc.yaml

# create the class
kubectl apply -f sc.yaml


