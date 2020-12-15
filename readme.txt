https://rancher.com/blog/2019/deploying-redis-cluster

$ kubectl apply -f redis-sts.yaml
configmap/redis-cluster created
statefulset.apps/redis-cluster created

$ kubectl apply -f redis-svc.yaml
service/redis-cluster created

$ kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
redis-cluster-0                    1/1     Running   0          7m
redis-cluster-1                    1/1     Running   0          7m
redis-cluster-2                    1/1     Running   0          6m
redis-cluster-3                    1/1     Running   0          6m
redis-cluster-4                    1/1     Running   0          6m
redis-cluster-5                    1/1     Running   0          5m

kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                          STORAGECLASS   REASON   AGE
pvc-ae61ad5c-f0a5-11e8-a6e0-42010aa40039   1Gi        RWO            Delete           Bound    default/data-redis-cluster-0   standard                7m
pvc-b74b6ef1-f0a5-11e8-a6e0-42010aa40039   1Gi        RWO            Delete           Bound    default/data-redis-cluster-1   standard                7m
pvc-c4f9b982-f0a5-11e8-a6e0-42010aa40039   1Gi        RWO            Delete           Bound    default/data-redis-cluster-2   standard                6m
pvc-cd7af12d-f0a5-11e8-a6e0-42010aa40039   1Gi        RWO            Delete           Bound    default/data-redis-cluster-3   standard                6m
pvc-d5bd0ad3-f0a5-11e8-a6e0-42010aa40039   1Gi        RWO            Delete           Bound    default/data-redis-cluster-4   standard                6m
pvc-e3206080-f0a5-11e8-a6e0-42010aa40039   1Gi        RWO            Delete           Bound    default/data-redis-cluster-5   standard                5m

# We can inspect any of the Pods to see its attached volume:
$ kubectl describe pods redis-cluster-0 
  Normal  SuccessfulAttachVolume  29m   attachdetach-controller       

# Deploy Redis Cluster
The next step is to form a Redis Cluster. To do this, we run the following command and type yes to accept the configuration. The first three nodes become masters, and the last three become slaves.

# Run the command to create the Cluster
Example: 
./redis-cli --cluster create 127.0.0.1:6001 127.0.0.1:6002 127.0.0.1:6003 127.0.0.1:6004 127.0.0.1:6005 127.0.0.1:6006 --cluster-replicas 1

# Run this command first
kubectl get pods -l app=redis-cluster -o jsonpath='{range.items[*]}{.status.podIP}:6379'

=> '172.17.0.7:6379'172.17.0.8:6379'172.17.0.9:6379'172.17.0.10:6379'172.17.0.11:6379'172.17.0.12:6379':6379'

# Run this final command
kubectl exec -it redis-cluster-0 -- redis-cli --cluster create 172.17.0.7:6379 172.17.0.8:6379 172.17.0.9:6379 172.17.0.10:6379 172.17.0.11:6379 172.17.0.12:6379 --cluster-replicas 1


# Verify Cluster Deployment
Check the cluster details and the role for each member.

$ kubectl exec -it redis-cluster-0 -- redis-cli cluster info

# Verify Cluster Roles
kubectl exec redis-cluster-0 -- redis-cli role
kubectl exec redis-cluster-1 -- redis-cli role
kubectl exec redis-cluster-2 -- redis-cli role
kubectl exec redis-cluster-3 -- redis-cli role
kubectl exec redis-cluster-4 -- redis-cli role
kubectl exec redis-cluster-5 -- redis-cli role

# Testing the Redis Cluster

kubectl apply -f app-deployment-service.yaml
service/hit-counter-lb created
deployment.apps/hit-counter-app created

# habilite o acesso externo a app
minikube service hit-counter-lb
|-----------|----------------|-------------|---------------------------|
| NAMESPACE |      NAME      | TARGET PORT |            URL            |
|-----------|----------------|-------------|---------------------------|
| default   | hit-counter-lb |          80 | http://192.168.49.2:30123 |
|-----------|----------------|-------------|---------------------------|
* Starting tunnel for service hit-counter-lb.
|-----------|----------------|-------------|------------------------|
| NAMESPACE |      NAME      | TARGET PORT |          URL           |
|-----------|----------------|-------------|------------------------|
| default   | hit-counter-lb |             | http://127.0.0.1:49654 |
|-----------|----------------|-------------|------------------------|
* Opening service default/hit-counter-lb in default browser...

#A mensagem abaixo deverá ser carregada
I have been hit 6 times since deployment.


# Simulate a Node Failure
kubectl delete pod redis-cluster-0
kubectl delete pod redis-cluster-1