# COMMON COMMANDS 
kubectl get deployments -o wide

kubectl get pods 
                --all-namespaces -o wide

kubectl exec --stdin --tty nginx-app-6f4c6d74d9-f7757 -- /bin/bash

kubectl -n kubernetes-dashboard create token admin-user

# GRACEFUL SHUTDOWN
```kubectl drain k8s-ha1  --ignore-daemonsets --delete-emptydir-data```


# POD MANAGEMENT 
kubectl delete --all bd -n openebs

kubectl delete pod  -l name=openebs-ndm -n openebs

kubectl delete pods -l app=my-app

kubectl delete pods -l app=my-app -n default

# SET DEFAULT STORAGE CLASEE ON CLUSTER
- list all storage class
kubectl get sc
- set default storage class
kubectl patch storageclass cstor-csi-disk  -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'

# INSTALL SPEED TEST
helm repo add TrueCharts https://charts.truecharts.org
helm repo update
helm install openspeedtest TrueCharts/openspeedtest

# cStore Troubleshooting 
https://openebs.io/docs/troubleshooting/cstor

# Block Devices
kubectl get bd -n openebs -o wide

# RESCAN openebs block devices
kubectl delete pod  -l name=openebs-ndm -n openebs

# CSTOR restart admission server
kubectl -n openebs get pods -o name | grep admission-server | xargs kubectl -n openebs delete

# CSTOR logs 
 kubectl logs -f -n openebs openebs-cstor-csi-node-q99fl cstor-csi-plugin

# CSTOR describe cvc 
kubectl describe cvc -n openebs

# CSTOR cStorVolumeReplica
kubectl get cvr -n openebs

# CSI CONTROLLER LOGS
kubectl logs openebs-cstor-csi-controller-0 -n openebs

# CSTOR cStor volume
kubectl get cstorvolume -n openebs

# CSTOR pool pods
kubectl get po -n openebs -l app=cstor-pool

# CSTOR cspi 
kubectl get cspi -n openebs

# CSTOR 
kubectl get cspc -n openebs

# DASHBOARD URL 
https://10.20.40.21:32592/#/workloads?namespace=default

kubectl -n kubernetes-dashboard create token admin-user


# INSTALLING CHARTS USING HELM
Before you install a chart it must not be on there already 
helm list -A 

If you want to remove OpenEBS you must follow these steps:

Delete all pvc and pv 
kubectl delete pvc -A --all
kubectl delete pv -A --all
kubectl delete sc -n openebs cstor-sc

AT THIS POINT,
you'll notice pods created to cleanup, they must finish 
confirm no pv pvc or sc remain
helm uninstall openebs -n openebs
wait again for pods to complete cleanup 
kubectl namespace delete openebs

# HELM INSTALLATIONS NLGKUBE
# GRAYLOG + MONGODB + ELASTICSEARCH 
1. 
helm install mongodb -n graylog bitnami/mongodb --set image.tag=4.2.21-debian-10-r8 --set architecture="replicaset"
2. 
helm install --namespace "graylog" elasticsearch elastic/elasticsearch
3. 
helm install --namespace "graylog" graylog kongz/graylog   --set tags.install-mongodb=false  --set tags.install-elasticsearch=false  --set graylog.mongodb.uri=mongodb://mongodb-arbiter-headless.graylog:27017/graylog?replicaSet=rs0   --set graylog.elasticsearch.hosts=http://elasticsearch-master.graylog:9200


# DELETE STUBBORN NAMESPACE
kubectl delete namespace [your-namespace] 

kubectl get namespace [your-namespace] -o json >tmp.json

remove finalizers from tmp.json 
start proxy kubectl proxy
then post that json to api
curl -k -H "Content-Type: application/json" -X PUT --data-binary @tmp.json http://127.0.0.1:8001/api/v1/namespaces/[your-namespace]/finalize


# EJECT A NODE GRACEFULLY
kubectl drain k8s-ha1 --ignore-daemonsets  --delete-emptydir-data
if there is data not replicated to other nods, this command will wait until the other nodes have replicas


# CVR uses targetip(target pod service ip) to connect to target pod. During restore, targetip is not set. This is to restore incremental backup after full backup restore.

Once restore completes, if incremental backups are there then those backups are also needs to be restored to retrieve all the data, targetip needs to be updated in replica.
steps:

To fetch targetip
kubectl get svc -n openebs <PV_NAME> -ojsonpath='{.spec.clusterIP}'
In your case, command will be
target_ip=kubectl get svc -n openebs pvc-9e222d37-984f-45ad-a42b-5fac7892b51f -ojsonpath='{.spec.clusterIP}'

Once you get targetip, you need to set it in all the replica of restored pv using following command:
kubectl exec -it <POOL_POD> -c cstor-pool -n openebs -- bash
zfs set io.openebs:targetip=<TARGET_IP> <POOL_NAME/VOLUME_NAME>
In your case, it will be

For CVR, pvc-9e222d37-984f-45ad-a42b-5fac7892b51f-cstor-disk-pool-eiq0
pool_pod=kubectl get pods -n openebs |grep cstor-disk-pool-eiq0 |awk -F ' ' '{print $1}'
kubectl exec -it $pool_pod -c cstor-pool -n openebs -- bash
POOL_VOLUME_NAME=zfs list |grep pvc-2d13e86e-d11c-40d4-947f-8b7f03ffa6d3 |awk -F ' ' '{print $1}'
`zfs set io.openebs:targetip=$target_ip $POOL_VOLUME_NAME

For CVR, pvc-9e222d37-984f-45ad-a42b-5fac7892b51f-cstor-disk-pool-una0
pool_pod=kubectl get pods -n openebs |grep cstor-disk-pool-una0 |awk -F ' ' '{print $1}'
kubectl exec -it $pool_pod -c cstor-pool -n openebs -- bash
POOL_VOLUME_NAME=zfs list |grep pvc-2d13e86e-d11c-40d4-947f-8b7f03ffa6d3 |awk -F ' ' '{print $1}'
`zfs set io.openebs:targetip=$target_ip $POOL_VOLUME_NAME

For CVR, pvc-9e222d37-984f-45ad-a42b-5fac7892b51f-cstor-disk-pool-7it6
pool_pod=kubectl get pods -n openebs |grep cstor-disk-pool-7it6 |awk -F ' ' '{print $1}'
kubectl exec -it $pool_pod -c cstor-pool -n openebs -- bash
POOL_VOLUME_NAME=zfs list |grep pvc-2d13e86e-d11c-40d4-947f-8b7f03ffa6d3 |awk -F ' ' '{print $1}'
`zfs set io.openebs:targetip=$target_ip $POOL_VOLUME_NAME
