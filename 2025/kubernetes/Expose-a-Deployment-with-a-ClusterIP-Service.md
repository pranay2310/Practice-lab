In this foundational lab you will learn how Kubernetes Services work by creating a Deployment and exposing it internally using a ClusterIP Service. ClusterIP is the default Service type and makes your application reachable from within the cluster. You will create a dedicated namespace, deploy a simple nginx web server, expose it with a ClusterIP Service, and finally verify connectivity using `kubectl port-forward` and `kubectl exec`.

-----------------------------------------------------------------------------
Create a Dedicated Namespace
Before deploying anything, create an isolated namespace called 
web-demo
✓
 to keep all lab resources organised.
1. Create the namespace by running:
kubectl create namespace web-demo
✓
2. Confirm the namespace exists and is Active:
kubectl get namespace web-demo
✓
You should see output similar to:
NAME       STATUS   AGE
web-demo   Active   5s
Copy
All subsequent resources in this lab will live in the 
web-demo
✓
 namespace.

 -----------------------------------------------------------------------------
Deploy the nginx Web Server
Create an nginx Deployment with **2 replicas** in the 
web-demo
✓
 namespace using a YAML manifest.
1. Create the file 
nginx-deployment.yaml
✓
 in the toolbox container:
cat <<'EOF' > nginx-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-web
  namespace: web-demo
  labels:
    app: nginx-web
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-web
  template:
    metadata:
      labels:
        app: nginx-web
    spec:
      containers:
        - name: nginx
          image: nginx:1.25-alpine
          ports:
            - containerPort: 80
EOF
Copy
2. Apply the manifest:
kubectl apply -f nginx-deployment.yaml
✓
3. Wait for both replicas to become Ready:
kubectl rollout status deployment/nginx-web -n web-demo
✓
4. Confirm the pods are Running:
kubectl get pods -n web-demo -l app=nginx-web

 -----------------------------------------------------------------------------
Expose the Deployment with a ClusterIP Service
Now expose the 
nginx-web
✓
 Deployment using a ClusterIP Service. A ClusterIP Service assigns a stable virtual IP address reachable only from within the cluster and load-balances traffic across all matching pods.
1. Create the file 
nginx-service.yaml
✓
:
cat <<'EOF' > nginx-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  namespace: web-demo
spec:
  type: ClusterIP
  selector:
    app: nginx-web
  ports:
    - name: http
      port: 80
      targetPort: 80
      protocol: TCP
EOF
Copy
2. Apply the Service manifest:
kubectl apply -f nginx-service.yaml
✓
3. Inspect the Service — note the 
CLUSTER-IP
✓
 value:
kubectl get service nginx-svc -n web-demo
✓
4. View the Endpoints to confirm the Service has discovered both pods:
kubectl get endpoints nginx-svc -n web-demo
✓
copy
You should see **two** endpoint IP:port pairs listed under 
ENDPOINTS
✓
.
 -----------------------------------------------------------------------------
Verify Connectivity via kubectl exec
Verify that the ClusterIP Service is reachable from **inside the cluster** by running 
curl
✓
 from one of the nginx pods against the Service DNS name.
Kubernetes automatically creates a DNS record for every Service in the format 
<service-name>.<namespace>.svc.cluster.local
✓
.
1. Get the name of one running pod:
kubectl get pods -n web-demo -l app=nginx-web -o name
✓
2. Run 
curl
✓
 against the Service DNS name from inside that pod (replace 
<pod-name>
✓
 with the actual pod name from step 1):
kubectl exec -n web-demo <pod-name> -- curl -s http://nginx-svc.web-demo.svc.cluster.local
✓
You should see the default nginx HTML welcome page in the output.
3. You can also use the short DNS name since the pod is in the same namespace:
kubectl exec -n web-demo <pod-name> -- curl -s http://nginx-svc
✓
**Tip:** If you prefer, use a one-liner that picks a pod name automatically:
POD=$(kubectl get pods -n web-demo -l app=nginx-web -o jsonpath='{.items[0].metadata.name}')
kubectl exec -n web-demo $POD -- curl -s http://nginx-svc.web-demo.svc.cluster.local
 -----------------------------------------------------------------------------
 Access the Service from the Toolbox via Port-Forward
ClusterIP Services are only reachable inside the cluster. To access the Service from your toolbox container (outside the cluster network), use 
kubectl port-forward
✓
.
1. Start port-forwarding the 
nginx-svc
✓
 Service to local port 
8080
✓
 in the **background**:
kubectl port-forward service/nginx-svc 8080:80 -n web-demo &
Copy
The 
&
✓
 runs the command in the background so you keep your shell prompt.
2. Wait a moment, then send a request to 
localhost:8080
✓
:
curl -s http://localhost:8080 | head -5
✓
You should see the first 5 lines of the nginx welcome page.
3. Bring the port-forward back to the foreground and stop it when done:
fg
✓
 then press 
Ctrl+C
✓
**Why this works:** 
kubectl port-forward
✓
 tunnels traffic from your local port through the Kubernetes API server into the cluster, forwarding it to the Service's ClusterIP and then to a pod.
Show hint