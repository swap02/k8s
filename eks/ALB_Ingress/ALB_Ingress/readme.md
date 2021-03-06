# Step 01: 
## Create Service Account
````bash
#List Service Accounts
kubectl get sa -n kube-system

#Create ClusterRole, ClusterRoleBinding & ServiceAccount
kubectl apply -f rbac-role.yaml

#List Service Accounts
kubectl get sa -n kube-system

#Describe Service Account alb-ingress-controller 
kubectl describe sa alb-ingress-controller -n kube-system
````

# Step 02: 
## Create IAM Policy for ALB Ingress Controller
 
| :point_up:    | Go to AWS console and create policy with ALBIngressControllerIAMPolicy.json |
|---------------|:----------------------------------------------------------------------------|


# Step 03
## Create an IAM role using eksctl

````bash
#Replaced region, name, cluster and policy arn (Policy arn we took note in step-03)
eksctl create iamserviceaccount \
    --region ap-south-1 \
    --name alb-ingress-controller \
    --namespace kube-system \
    --cluster formaceksdemo \
    --attach-policy-arn arn:aws:iam::XXXXXXXXXXXX:policy/ALBIngressControllerIAMPolicy \
    --override-existing-serviceaccounts \
    --approve
 ````
 
| :memo:        | Verify IAM Account       |
|---------------|:------------------------|
 
 ````bash
 # Get IAM Service Account
eksctl  get iamserviceaccount --cluster formaceksdemo --region ap-south-1
 ````
 
| :memo:        | Verify k8s Service Account|
|---------------|:------------------------|

````bash
# Describe Service Account alb-ingress-controller 
kubectl describe sa alb-ingress-controller -n kube-system
````


# Step-04: Deploy ALB Ingress Controller

````bash
# Deploy ALB Ingress Controller
kubectl apply -f alb-ingress-controller.yaml

# Verify Deployment
kubectl get deploy -n kube-system
````

# Step-05
1. Edit ALB Ingress Controller manifest and add clustername field - --cluster-name=formaceksdemo

````bash
# Edit Deployment
kubectl edit deployment.apps/alb-ingress-controller -n kube-system

# Template file  
    spec:
      containers:
      - args:
        - --ingress-class=alb
        - --cluster-name=cluster-name

# Replaced cluster-name with our cluster-name eksdemo1
    spec:
      containers:
      - args:
        - --ingress-class=alb
        - --cluster-name=formaceksdemo
 ````
 
 # Step-06: Verify our ALB Ingress Controller is running
 
| :memo:        | Verify if alb-ingress-controller pod is running|
|---------------|:-----------------------------------------------|
````bash
kubectl get pods -n kube-system
`````

| :memo:        | Verify logs|
|---------------|:-----------|
````bash
kubectl logs -f $(kubectl get po -n kube-system | egrep -o 'alb-ingress-controller-[A-Za-z0-9-]+') -n kube-system
 ````
