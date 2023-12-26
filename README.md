# kubernetes-tutorial

Some simple example of kubernetes

# Docs:

[Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)

# Install minikube

- [Install](https://minikube.sigs.k8s.io/docs/handbook/kubectl/)
- curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
- sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Minikube:

- minikube config set memory 16384 8192
- minikube config set cpus 6
- minikube start
- minikube dashboard
- minikube service fleetman-webapp --url
- minikube service fleetman-queue --url

# Kubectl:

- kubectl cluster-info
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
- kubectl get pv
- kubectl get pvc

# Networking inside a pod

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

# ELK

## Install Helm

- curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
- chmod 700 get_helm.sh
- ./get_helm.sh

## Download helm packages

- Download logstash [logstash](https://artifacthub.io/packages/helm/elastic/logstash)
- Download filebeat [filebeat](https://artifacthub.io/packages/helm/elastic/filebeat)
- Download elasticsearch [elasticsearch](https://artifacthub.io/packages/helm/elastic/elasticsearch)
- Download kibana [kibana](https://artifacthub.io/packages/helm/elastic/kibana)

## Edit values

logstash/values.yaml:

- replace "logstashPipeline" as follows:

```yml
logstashPipeline:
  logstash.conf: |
    input {
      beats {
        port => 5044
      }
    }
    output { elasticsearch { hosts => "http://elasticsearch-master:9200" } }
```

- and edit "service" as follow:

```yml
service:
  annotations: {}
  type: ClusterIP
  loadBalancerIP: ""
  ports:
    - name: beats
      port: 5044
      protocol: TCP
      targetPort: 5044
    - name: http
      port: 8080
      protocol: TCP
      targetPort: 8080
```

filebeat/values.yaml:

- replace "filebeatConfig" as follows:

```yml
filebeatConfig:
  filebeat.yml: |
    filebeat.inputs:
    - type: container
      paths:
        - /var/log/containers/*.log
      processors:
      - add_kubernetes_metadata:
          host: ${NODE_NAME}
          matchers:
          - logs_path:
              logs_path: "/var/log/containers/"

    output.logstash:
      hosts: ["logstash-logstash:5044"]

# output.elasticsearch:
#   host: '${NODE_NAME}'
#   hosts: '["https://${ELASTICSEARCH_HOSTS:elasticsearch-master:9200}"]'
#   username: '${ELASTICSEARCH_USERNAME}'
#   password: '${ELASTICSEARCH_PASSWORD}'
#   protocol: https
#   ssl.certificate_authorities: ["/usr/share/filebeat/certs/ca.crt"]
```

elasticsearch/values.yaml:

- change "antiAffinity" to "soft" value

## Install packages

- helm install elasticsearch ./elasticsearch
- helm install filebeat ./filebeat
- helm install logstash ./logstash
- helm install kibana ./kibana

## Responses

- elasticsearch

```yml
NAME: elasticsearch
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Watch all cluster members come up.
  $ kubectl get pods --namespace=default -l app=elasticsearch-master -w
2. Retrieve elastic user's password.
  $ kubectl get secrets --namespace=default elasticsearch-master-credentials -ojsonpath='{.data.password}' | base64 -d
3. Test cluster health using Helm test.
  $ helm --namespace=default test elasticsearch
```

- kibana

```yml
NAME: kibana
LAST DEPLOYED: Mon Dec 25 16:44:41 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
1. Watch all containers come up.
  $ kubectl get pods --namespace=default -l release=kibana -w
2. Retrieve the elastic user's password.
  $ kubectl get secrets --namespace=default elasticsearch-master-credentials -ojsonpath='{.data.password}' | base64 -d
3. Retrieve the kibana service account token.
  $ kubectl get secrets --namespace=default kibana-kibana-es-token -ojsonpath='{.data.token}' | base64 -d
```

## Port forward to run kibana on browser

- kubectl port-forward svc/kibana-kibana 5601:5601

## Obtain password to login

- username: elastic
- password: will be generated with following command:

```bash
kubectl get secrets --namespace=default elasticsearch-master-credentials -ojsonpath='{.data.password}' | base64 -d
```

## Some get commands

- kubectl get jobs
- kubectl get serviceaccount
- kubectl get configmap
- kubectl get secrets
- kubectl get roles
- kubectl get deployments
- kubectl get pods
- kubectl get services
- kubectl get pv
- kubectl get pvc
- kubectl get ingress
- kubectl get rolebindings
- kubectl get clusterroles
- kubectl get clusterrolebindings

## Delete parts relative to the kibana

- kubectl delete jobs pre-install-kibana-kibana
- kubectl delete serviceaccount pre-install-kibana-kibana
- kubectl delete configmap kibana-kibana-helm-scripts
- kubectl delete roles pre-install-kibana-kibana
- kubectl delete rolebindings pre-install-kibana-kibana
