### TODOS
  ➜ [Create a Rancher Server with a single node cluster (some experiments require multi node)](#create-a-rancher-server)

  ➜ [Configure rancher cluster](#configure-rancher-cluster)

  ➜ [Deploy the demo microservice](#deploy-the-microservice)

  ➜ [Run experiments and observe (repeat steps for each experiment](#run-experiments-and-observe)

### Requirements
- Kubernetes/kubectl
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- [Vagrant](https://www.vagrantup.com/docs/installation)
- plugins for Vagrant

### Create a Rancher Server 
Note: I am using ranchers getting stared  with Vagrant configuration along with a demo [microservice](https://github.com/microservices-demo/microservices-demo)): 
  ```BASH
     https://github.com/rancher/quickstart/tree/master/vagrant
     https://rancher.com/docs/rancher/v2.x/en/quick-start-guide/deployment/quickstart-vagrant/
  ```


1. Install plugins to create VirtualBox VMs
   
    ```BASH
    vagrant plugin install vagrant-vboxmanage

    vagrant plugin install vagrant-vbguest
    ```  

2. Create the environment 
   
    ```BASH
    vagrant up --provider=virtualbox
    ```
3. Navigate to the browser 
   
    ```BASH
    https://172.22.101.101
    #username|password: admin|admin
    ```


### Configure Rancher Cluster 

1. Change the name of the cluster (optional)
   
2. Scroll to the bottom and check `etcd`,`Control Plane`, and `Worker`.
   
3. Copy the contents of the clipboard then hit `Save`
   
4. SSH into vagrant
    ```BASH
    vagrant ssh server-01
    ```   
5. Paste contents of clipboard into server terminal
   
6. Open a new terminal on the server and watch the container
    ```BASH
    docker ps
    docker container logs <name> --follow
    ```
7. Navigate back to browser and wait for cluster to have an active status 
   
8. Once created click on the cluster to access the dashboard
   
9.  Under `Kubeconfig File` copy the file contents and put it in a file under this dir
    ```BASH
    touch kube.yaml
    vim kube.yaml 
    #insert(i),paste,save and quite (:wq)
    ```

### Deploy the Microservice 

1. Check you have access to rancher on your local machine, deploy demo microservice, verify pods are running
   
    ```BASH
    kubectl --kubeconfig kube.yaml get pods --namespace=cattle-system
    #deploy
    kubectl --kubeconfig kube.yaml create -f sock-shop.yaml
    #verify - wait for running status
    kubectl --kubeconfig kube.yaml get pods -n sock-shop
    ```
2. Deploy Litmus ChaosOperator
   
    ```BASH
    kubectl --kubeconfig kube.yaml apply -f https://litmuschaos.github.io/litmus/litmus-operator-v1.9.0.yaml
    ```
3. Install Litmus Experiments
   
    ```BASH
    curl -sL https://github.com/litmuschaos/chaos-charts/archive/1.9.0.tar.gz -o litmus.tar.gz
    tar -zxvf litmus.tar.gz
    rm litmus.tar.gz
    find chaos-charts-1.9.0 -name experiments.yaml | grep generic | xargs kubectl --kubeconfig kube.yaml apply -n sock-shop -f
    ```
4. Create Service Account
   
    ```BASH
    kubectl  --kubeconfig kube.yaml create -f rbac.yaml
    ```
5. Access url of sock shop microservice

   Get front-end deployment port info:
    ```BASH
    kubectl  --kubeconfig kube.yaml get deploy front-end -n sock-shop -o jsonpath='{.spec.template.spec.containers[?(@.name == "front-end")].ports[0].containerPort}'
    ```
   Set port forwarding
    ```BASH
    kubectl  --kubeconfig kube.yaml port-forward deploy/front-end -n sock-shop 3000:8079
    #browser address: 127.0.0.1:3000
   ```
### Run Experiments and Observe

1. Delete any existing Chaos engines in the namespace

    ```BASH
    kubectl --kubeconfig kube.yaml delete chaosengine rancher-chaos -n sock-shop
    ```

2. Run the experiment 

   ```BASH
    kubectl --kubeconfig kube.yaml create -f litmus/pod-memory-hog.yaml -n sock-shop
    ```
3. Observe Results (Takes a few seconds for command to turnover, results initially in `await` state)

    ```BASH
   kubectl --kubeconfig kube.yaml get pods -n sock-shop --watch
    kubectl --kubeconfig kube.yaml describe chaosengine rancher-chaos -n sock-shop
    kubectl --kubeconfig kube.yaml describe chaosresult rancher-chaos-pod-memory-hog -n sock-shop
    #to save results to file: 
    kubectl --kubeconfig kube.yaml describe chaosengine rancher-chaos -n sock-shop > litmus-results/chaosengine-pod-memory-hog.txt

    kubectl --kubeconfig kube.yaml describe chaosresult rancher-chaos-pod-memory-hog -n sock-shop > litmus-results/chaosresult-pod-memory-hog.txt
    ```
4. Uninstall

    ```BASH
    kubectl --kubeconfig kube.yaml delete -f https://litmuschaos.github.io/litmus/litmus-operator-v1.9.0.yaml
    kubectl delete chaosengine --all -n sock-shop
    kubectl --kubeconfig kube.yaml delete chaosengine --all -n litmus
    kubectl --kubeconfig kube.yaml delete namespaces sock-shop
    ```

