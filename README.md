### Example Basic for Dapr ### 

1. Install k0s

curl -sSLf https://get.k0s.sh | sudo sh

k0s install controller --enable-worker

k0s start

mkdir -o /root/.kube

cp /var/lib/k0s/pki/admin.conf /root/.kube/config

2. Install helm

curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh

3. Install dapr

helm install dapr dapr/dapr   --version 1.14.5   --namespace dapr-system   --create-namespace   --set global.ha.enabled=false --set dapr_scheduler.cluster.storageSize=16Gi --set dapr_scheduler.etcdSpaceQuota=16Gi



4. Execute files

docker build -t NAME_REPO/dapr-api:0.1 .
docker push NAME_REPO/dapr-api:0.1

mkdir -p /mnt/data/dapr-scheduler-0
chmod 777 /mnt/data/dapr-scheduler-0

kubectl apply -f pv.yml

kubectl get pods -n dapr-system

kubectl apply -f deploy.yml 

