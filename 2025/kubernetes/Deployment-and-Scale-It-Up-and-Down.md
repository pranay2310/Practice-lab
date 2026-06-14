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

--------------------------------------------------------

--------------------------------------------------------

--------------------------------------------------------

--------------------------------------------------------