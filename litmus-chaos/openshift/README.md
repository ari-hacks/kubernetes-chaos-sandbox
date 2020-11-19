#### TODOS
 - Create a single node OKD cluster with minishift (some experiments require multi node)
 - Deploy the demo [mircoservice](https://github.com/GoogleCloudPlatform/microservices-demo)
 - Run experiments

#### Requirements
- Kubernetes/kubectl 
- VirtualBox 
- minishift
- [`hyperkit` and `docker-machine-driver-hyperkit`](https://docs.okd.io/3.11/minishift/getting-started/setting-up-virtualization-environment.html)

#### Create a single node OKD cluster with minishift

1. Run minishift start command 
```BASH
minishift start
#if you run into errors skip startup checks
minishift config set skip-startup-checks true
```  

2. Display path needed for  `oc` binary
```BASH
minishift oc-env
#add output path to PATH environment variable
eval $(minishift oc-env)
#admin loging 
oc login -u system:admin
```
3. `minishift start` outputs info to connect to the web console. Follow the prompt to get to the web dashboard
Note: OKD prompts you to create a project, these projects act as clusters/namespaces in Kubernetes. Create a new project in the dashboard named `okd-cluster` 



#### Deploy the demo mircoservice and install Litmus

1. In the terminal set the context to the project you created
```BASH
oc project okd-cluster
```
2. Deploy the microservice to the cluster 
```BASH
oc apply -f gcp-microservice.yaml
#check status
oc get pods -n okd-cluster
```
3. Access web frontend with external ip
```BASH
oc port-forward deployment/frontend 8080:8080
```

4. Deploy Litmus ChaosOperator
```BASH
oc apply -f https://litmuschaos.github.io/litmus/litmus-operator-v1.9.0.yaml
```
5. Install Litmus Experiments
```BASH
 curl -sL https://github.com/litmuschaos/chaos-charts/archive/1.9.0.tar.gz -o litmus.tar.gz
 tar -zxvf litmus.tar.gz
 rm litmus.tar.gz
find chaos-charts-1.9.0 -name experiments.yaml | grep generic | xargs oc apply -n okd-cluster -f
```
6. Create Service Account
```BASH
 oc create -f rbac.yaml
```

#### Run experiments and observe (repeat steps for each experiment)

1. Delete any existing Chaos engines in the namespace
```BASH
  oc delete chaosengine okd-chaos -n okd-cluster
```

2. Run the experiment and watch
```BASH
  oc create -f litmus/pod-delete.yaml -n okd-cluster
    
  oc get pods -n okd-cluster --watch
```
3. Observe Results (Takes a few seconds for command to turnover, results initially in `await` state)
```BASH
 oc describe chaosengine okd-chaos -n okd-cluster

 oc describe chaosresult okd-chaos-pod-delete -n okd-cluster

 #to save results to file: 
 oc describe chaosengine okd-chaos -n okd-cluster > litmus-results/chaosengine-pod-delete.txt

 oc describe chaosresult okd-chaos-pod-delete -n okd-cluster > litmus-results/chaosresult-pod-delete.txt
```

4. Uninstall and clean up
```BASH
  oc delete -f https://litmuschaos.github.io/litmus/litmus-operator-v1.9.0.yaml

  oc delete chaosengine --all -n okd-cluster

  oc delete chaosengine --all -n litmus

  oc delete namespaces okd-cluster

  minishift stop

  minishift delete 
```

