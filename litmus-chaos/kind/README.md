#### TODOS
 - Create a kind cluster
 - Deploy the demo mircoservice [sock shop](https://github.com/microservices-demo/microservices-demo)
 - Run experiments

#### Requirements
- Kubernetes/kubectl  
- [kind](https://kind.sigs.k8s.io/docs/user/quick-start/)

#### Setting up kind cluster and setup litmus

1. Verify kind installation
    ```BASH
    kind version
    ```
2. Create kind cluster
   ```BASH
   kind create cluster --config kind-config.yaml 
   ```
3. Set context and verify nodes
    ```BASH
    #specify cluster name as a context
    kubectl cluster-info --context kind-kind
    #verify
    kubectl get nodes
    ```

4. Deploy demo microservice and verify they are running 
   ```BASH
    #deploy
    kubectl create -f sock-shop.yaml
    #verify - wait for running status
    kubectl get pods -n sock-shop
    ```
5. Deploy Litmus ChaosOperator
    ```BASH
    kubectl apply -f https://litmuschaos.github.io/litmus/litmus-operator-v1.9.0.yaml
    ```
6. Install Litmus Experiments 
    ```BASH
    curl -sL https://github.com/litmuschaos/chaos-charts/archive/1.9.0.tar.gz -o litmus.tar.gz
    tar -zxvf litmus.tar.gz
    rm litmus.tar.gz
    find chaos-charts-1.9.0 -name experiments.yaml | grep generic | xargs kubectl apply -n sock-shop -f
    ```
7. Create Service Account
   ```BASH
   kubectl create -f rbac.yaml
   ```

####  Access url of sock shop microservice

1. Get front end deployment
    ```BASH
     kubectl get deploy front-end -n sock-shop -o jsonpath='{.spec.template.spec.containers[?(@.name == "front-end")].ports[0].containerPort}'
    ```

2. Set port forwarding 
   ```BASH
    kubectl port-forward deploy/front-end -n sock-shop 3000:8079
    #browser address: 127.0.0.1:3000
    ```

#### Run experiments and observe (repeat steps for each experiment)

1. Delete any existing Chaos engines in the namespace
    ```BASH
    kubectl delete chaosengine kind-chaos -n sock-shop
    ```

2. Run the experiment 
   ```BASH
   kubectl create -f litmus/container-kill.yaml -n sock-shop
   ```
3. Observe Results (Takes a few seconds for command to turnover, results initially in `await` state)
     ```BASH
    kubectl get pods -n sock-shop --watch
     kubectl describe chaosengine kind-chaos -n sock-shop
    kubectl describe chaosresult kind-chaos-container-kill -n sock-shop
     #to save results to file: 
    kubectl describe chaosengine kind-chaos -n sock-shop > chaosengine.txt
    kubectl describe chaosresult kind-chaos-container-kill -n sock-shop > chaosresult.txt
    ```
 4. Uninstall
    ```BASH
    kubectl delete -f https://litmuschaos.github.io/litmus/litmus-operator-v1.9.0.yaml
    kubectl delete chaosengine --all -n sock-shop
    kubectl delete chaosengine --all -n litmus
    kubectl delete namespaces sock-shop
    ```

