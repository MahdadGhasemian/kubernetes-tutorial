# kubernetes-tutorial
Some simple example of kubernetes

# kubectl:

- kubectl apply -f .
- kubectl get all
- kubectl describe rs REPLICASET-NAME
- kubectl rollout status deployment DEPLOYMENT-NAME
- kubectl rollout history deploy DEPLOYMENT-NAME
- kubectl rollout undo deploy DEPLOYMENT-NAME
- kubectl get ns
- kubectl get all -n kube-system
- kubectl get all -n kube-public
- kubectl logs POD-NAME
- kubectl logs -f POD-NAME

# minikube:
- minikube service fleetman-webapp --url
- minikube service fleetman-queue --url

# networking inside a pod
- kubectl exec -it webapp-random-name sh
```bash
/ # cat /etc/resolv.conf
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5

/ # nslookup database
nslookup: can not resolve (null): Name does not resolve

Name:      database
Address 1: 10.108.106.120 database.default.svc.cluster.local
```

Domain Name : "database"
Fully Qualified Domain Name: "database.default.svc.cluster.local"
