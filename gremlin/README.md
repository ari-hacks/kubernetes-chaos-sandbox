#### TODOS
 - Create a HA production ready cluster with [Kubespray](https://github.com/kubernetes-sigs/kubespray)
 - Deploy the demo [mircoservice](https://github.com/GoogleCloudPlatform/microservices-demo)
 - Setup and run Gremlin 

#### Requirements
- Kubernetes/kubectl 
- Python3
- pip3
- VirtualBox 
- [Gremlin account](https://app.gremlin.com/signup)

#### Deploy a HA production ready cluster with Kubespray

Note: This repo uses [kubespray](https://github.com/kubernetes-sigs/kubespray) 
1. clone the repo and cd into it 
   ```BASH
    https://github.com/kubernetes-sigs/kubespray.git
    cd kubespray 
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
    #modify: group_vars/k8s-cluster/k8s-cluster.yml to
    kubeconfig_localhost: true
    #navigate to artifacts directory
    #/inventory/sample/artifacts
    cp admin.conf ~/.kube/config
    ```
5. Check nodes are up
    ```BASH
    kubectl get nodes

    kubectl get pods --all-namespaces
    ```


#### Deploy the demo mircoservice

1. Navigate to the base directory `/gremlin`

2. Create a namespace for the mircoservice 
    ```BASH
    kubectl create namespace sock-shop
    ```
3. Deploy it
    ```BASH
    kubectl apply -f sock-shop.yaml
    ```
4. Check pods are running
    ```BASH 
    kubectl get pods --namespace sock-shop
    #or
    kubectl get pods --namespace sock-shop --watch
    ```
5. Get front deployment
   ```BASH
   kubectl get deploy front-end -n sock-shop -o jsonpath='{.spec.template.spec.containers[?(@.name == "front-end")].ports[0].containerPort}'

   kubectl port-forward deploy/front-end -n sock-shop 3000:8079
   ```
#### Setup Gremlin

1. Once logged in get the team id and secret key from the [Teams page]([Setup](https://app.gremlin.com/settings/teams)) and download


2. Install the Gremlin client with helm
   ```BASH
    #NOTE:first change the env variables to match your team credentials
    helm repo add gremlin https://helm.gremlin.com
    ```
3. Create a namespace for the client
    ```BASH
    kubectl create namespace gremlin
    ```
4. Run to install (replace credentials)
   ```BASH
   export GREMLIN_TEAM_ID=<team-id>
   export GREMLIN_CLUSTER_ID=<cluster-di> #can be anything
   export GREMLIN_TEAM_SECRET=<team-scale>

   helm install gremlin gremlin/gremlin \
    --namespace gremlin \
    --set gremlin.secret.managed=true \
    --set gremlin.secret.type=secret \
    --set gremlin.secret.teamID=$GREMLIN_TEAM_ID \
    --set gremlin.secret.clusterID=$GREMLIN_CLUSTER_ID \
    --set gremlin.secret.teamSecret=$GREMLIN_TEAM_SECRET
   ```
5. In gremlin you can check and run attacks from the intuitive dashboard
   ```BASH
   https://app.gremlin.com/clients/hosts
   ```
    ### or use the [API](https://app.gremlin.com/api#/attacks/createhttps://app.gremlin.com/api#/attacks/create)

6. Clean up
    ```BASH
    helm uninstall -n gremlin gremlin
    vagrant destroy -f
    ```

#### Resources

[Official Gremlin docs](https://www.gremlin.com/community/tutorials/how-to-install-and-use-gremlin-with-kubernetes/)