### TODOS
 ➜ [Create a HA production ready cluster with Kubespray](#create-kubespray-cluster)

 ➜ [Deploy the demo microservice](#deploy-demo-microservice)

 ➜ [Deploy Chaos Mesh](#deploy-chaos-mesh)

 ➜ [Setup and run experiments (repeat steps for each experiment)](#setup-and-run-experiments)

### Requirements
- Kubernetes/kubectl 
- Python3
- pip3
- VirtualBox 

Note: This repo uses [kubespray](https://github.com/kubernetes-sigs/kubespray) and this demo [microservice](https://github.com/GoogleCloudPlatform/microservices-demo)

### Create Kubespray Cluster
Note: This repo uses [kubespray](https://github.com/kubernetes-sigs/kubespray) 
1. clone the repo and cd into it 
   
   ```BASH
    https://github.com/kubernetes-sigs/kubespray.git
    cd kubespray 
    #modify: group_vars/k8s-cluster/k8s-cluster.yml to
    kubeconfig_localhost: true
   ```
2. Install dependencies
   
    ```BASH
    pip3 install -r requirements.txt
    ```  

3. Bring up vagrant (this can take awhile)
   
    ```BASH
    vagrant up
    ```
4. Setup access to the cluster globally
   
    ```BASH
    #navigate to artifacts directory
    #/inventory/sample/artifacts
    cp admin.conf ~/.kube/config
    ```
5. Check nodes are up
   
    ```BASH
    kubectl get nodes

    kubectl get pods --all-namespaces
    ```


### Deploy Demo Microservice

1. Create a namespace for the microservice 
   
    ```BASH
    kubectl create namespace sock-shop
    ```
2. Deploy it
   
    ```BASH
    kubectl apply -f sock-shop.yaml
    ```
3. Check pods are running
   
    ```BASH 
    kubectl get pods --namespace sock-shop
    #or
    kubectl get pods --namespace sock-shop --watch
    ```
4. Get front-end deployment port info and setup port forwarding
   
   ```BASH
   kubectl get deploy front-end -n sock-shop -o jsonpath='{.spec.template.spec.containers[?(@.name == "front-end")].ports[0].containerPort}'

   kubectl port-forward deploy/front-end -n sock-shop 3000:8079
   #127.0.0.1:3000
   ```
### Deploy Chaos Mesh
1. Clone the repo in `/chaos-mesh`
   
   ```BASH
   https://github.com/chaos-mesh/chaos-mesh.git
   ```

2. Install and create chaos mesh
   
   ```BASH
   kubectl apply -f ./chaos-mesh/manifests/crd.yaml

   kubectl create ns chaos-mesh
    helm upgrade --install chaos-mesh ./chaos-mesh/helm/chaos-mesh \
    -n chaos-mesh \
    --set dashboard.create=true
   ```

3. Check pods are running and CRDs are present
   
   ```BASH
   kubectl get pods --namespace chaos-mesh
   kubectl get crds
   ```
4. Get chaos-dashboard port info
   
   ```BASH
    kubectl get deploy chaos-dashboard -n chaos-mesh -o=jsonpath="{.spec.template.spec.containers[0].ports[0].containerPort}{'\n'}"
   ```
5. Expose port
   
   ```BASH
   kubectl port-forward -n chaos-mesh svc/chaos-dashboard 2333:2333
   #127.0.0.1:2333
   ```
   
### Setup and run Experiments

1. Run experiments via yaml config
   
   ```BASH
    kubectl apply -f chaos-experiments/pod-kill.yaml
    ```

2. Monitor results

   ```BASH
   kubectl describe podchaos -n chaos-mesh

   http://localhost:2333
   ```

3. Clean up
   
    ```BASH
    kubectl delete -f chaos-experiments/pod-kill.yaml -n chaos-mesh
    helm delete chaos-mesh -n chaos-mesh
    kubectl delete namespace sock-shop
    vagrant halt
    vagrant destroy -f
    ```


### Resources

[Official Chaos mesh docs](https://chaos-mesh.org/docs/user_guides/run_chaos_experiment)

[Blog](https://dev.to/craigmorten/k8s-chaos-dive-2-chaos-mesh-part-1-2i96)
