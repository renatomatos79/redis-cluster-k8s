Clone this repository in your local machine:
cd c:\temp
git clone https://github.com/renatomatos79/redis-cluster-k8s.git
cd redis-cluster-k8s

And then, run the commands below in order to create the redis PODs and expose them using a service name redis-cluster:

1) kubectl apply -f redis-sts.yaml
2) kubectl apply -f redis-svc.yaml

# Run this command to get all IPs and Ports
kubectl get pods -l app=redis-cluster -o jsonpath='{range.items[*]}{.status.podIP}:6379'

# You will see something like this:
=> '172.17.0.7:6379'172.17.0.8:6379'172.17.0.9:6379'172.17.0.10:6379'172.17.0.11:6379'172.17.0.12:6379':6379'

# Now, remove the first and last char '

=> 172.17.0.7:6379'172.17.0.8:6379'172.17.0.9:6379'172.17.0.10:6379'172.17.0.11:6379'172.17.0.12:6379':6379

# And then, replace the char ' using a blank space

=> 172.17.0.7:6379 172.17.0.8:6379 172.17.0.9:6379 172.17.0.10:6379 172.17.0.11:6379 172.17.0.12:6379

3) Finally, we are able to run this final command. After that, in the command prompt answer YES to create the cluster
kubectl exec -it redis-cluster-0 -- redis-cli --cluster create 172.17.0.7:6379 172.17.0.8:6379 172.17.0.9:6379 172.17.0.10:6379 172.17.0.11:6379 172.17.0.12:6379 --cluster-replicas 1

# Let's verify the Cluster Deployment
kubectl exec -it redis-cluster-0 -- redis-cli cluster info

# Verify Cluster Roles
kubectl exec redis-cluster-0 -- redis-cli role
kubectl exec redis-cluster-1 -- redis-cli role
kubectl exec redis-cluster-2 -- redis-cli role
kubectl exec redis-cluster-3 -- redis-cli role
kubectl exec redis-cluster-4 -- redis-cli role
kubectl exec redis-cluster-5 -- redis-cli role

# Testing the Redis Cluster
kubectl apply -f app-deployment-service.yaml

# Enable the external access for your app 
minikube service hit-counter-lb

.... Opening service default/hit-counter-lb in default browser...

# For now you can verify the counter each time you refresh the page
I have been hit 1 times since deployment.

# Simulate a Node Failure
kubectl delete pod redis-cluster-0
kubectl delete pod redis-cluster-1

Reference:
https://rancher.com/blog/2019/deploying-redis-cluster

The end!
