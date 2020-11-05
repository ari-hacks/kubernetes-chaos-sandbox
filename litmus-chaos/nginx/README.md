#### TODOS
- Install Litmus
- Create nginx server with docker image 
- Run container-kill experiment on pod 

#### Requirements
- Kubernetes/kubectl  

#### Setting up pod 

1. Create an nginx namepsace 
```BASH
 kubectl create namespace nginx
```
2. Create deployment with nginx image
```BASH
 kubectl create deployment nginx --image=nginx -n nginx
```
3. Check pod was created successfully and has a Ready and Running status 
```BASH
 kubectl get pods --all-namespaces
```
4. Create service
```BASH
 kubectl create service nodeport nginx --tcp=80:80 -n nginx
```
5. Check service was created successfully
```BASH
 kubectl get svc -n nginx
```

#### Install Litmus 

1. Apply the LitmusChaos Operator manifest
```BASH 
 kubectl apply -f https://litmuschaos.github.io/litmus/litmus-operator-v1.9.0.yaml
```
2. Verify Litmus installation 
```BASH
#ChaosOperator running 
  kubectl get pods -n litmus
#Chaos CRDs installed 
  kubectl get crds | grep chaos
#Chaos resources created  
  kubectl api-resources | grep chaos
```
3. Install Experiments
```BASH 
 kubectl apply -f https://hub.litmuschaos.io/api/chaos/1.9.0\?file\=charts/generic/experiments.yaml -n nginx
#verify experiments are installed
  kubectl get chaosexperiments -n nginx
```

4. Setup Service Account
```BASH
#create rbac.yaml file.yaml and apply it 
 kubectl apply -f rbac.yaml
#annotate application
 kubectl annotate deploy/nginx litmuschaos.io/chaos="true" -n nginx
```
5. Prepare Chaos Engine and run it
```BASH
#create chaosengine.yaml and apply manifest
 kubectl apply -f chaosengine.yaml
```
6. Observe Results (Takes a few seconds for command to turnover, results initially in `await` state)
```BASH
 kubectl describe chaosengine nginx-chaos -n nginx
 kubectl describe chaosresult nginx-chaos-container-kill -n nginx
 #to save results to file: 
 kubectl describe chaosengine nginx-chaos -n nginx > chaosengine.txt
 kubectl describe chaosresult nginx-chaos-container-kill -n nginx > chaosresult.txt
```
7. Uninstall
```BASH
  kubectl delete -f https://litmuschaos.github.io/litmus/litmus-operator-v1.9.0.yaml
  kubectl delete chaosengine --all -n nginx
  kubectl delete chaosengine --all -n litmus
```