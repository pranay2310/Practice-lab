TASK 1 OF 5
Done
15 pts
Create a Namespace for the Lab
Namespaces let you logically isolate resources within a cluster. Before creating any workloads, create a namespace called 
web-lab
✓
 that will hold everything you build in this challenge.
1. Create the namespace by running:
kubectl create namespace web-lab
✓
2. Confirm it was created:
kubectl get namespace web-lab
✓
You should see the namespace listed with a STATUS of **Active**.

--------------------------------------------------------
Create an nginx Deployment
Now create an nginx Deployment inside the 
web-lab
✓
 namespace. You will write a YAML manifest and apply it with 
kubectl apply
✓
.
1. Inside your toolbox container, create the manifest file:
cat <<'EOF' > nginx-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
  namespace: web-lab
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
EOF
Copy
2. Apply the manifest:
kubectl apply -f nginx-deployment.yaml
✓
3. Watch the Deployment roll out:
kubectl rollout status deployment/nginx-deploy -n web-lab
✓
Wait until you see **successfully rolled out**.
--------------------------------------------------------
Inspect the Deployment, ReplicaSet, and Pods
Before scaling, take time to understand the objects Kubernetes created for you.
1. View the Deployment summary:
kubectl get deployment nginx-deploy -n web-lab
✓
2. View the ReplicaSet that the Deployment created:
kubectl get replicaset -n web-lab
✓
3. List the individual Pods and note their names:
kubectl get pods -n web-lab -o wide
✓
4. Describe the Deployment to see events and replica counts:
kubectl describe deployment nginx-deploy -n web-lab
✓
Notice the **READY**, **UP-TO-DATE**, and **AVAILABLE** columns in the Deployment output. You should see 
2/2
✓
 ready. Also observe that the ReplicaSet name is the Deployment name with a hash suffix, and each Pod name adds another hash on top of that.
Verify that exactly **2** pods are in the 
Running
✓
 phase before moving on.
--------------------------------------------------------
Scale the Deployment Up to 5 Replicas
Kubernetes makes it trivial to scale a Deployment. You can use the imperative 
kubectl scale
✓
 command or patch the manifest and re-apply it. In this task use the imperative command.
1. Scale the Deployment up to 5 replicas:
kubectl scale deployment nginx-deploy --replicas=5 -n web-lab
✓
2. Watch the new pods appear in real time (press Ctrl+C to stop watching):
kubectl get pods -n web-lab -w
✓
3. Once all pods are ready, confirm the Deployment status:
kubectl get deployment nginx-deploy -n web-lab
✓
copy
You should see **5/5** in the READY column. Kubernetes created 3 additional pods to match your new desired state — this is the reconciliation loop in action.
--------------------------------------------------------

--------------------------------------------------------

--------------------------------------------------------