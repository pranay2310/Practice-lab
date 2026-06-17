In this hands-on lab you will deploy a simple web application to a Kubernetes cluster and expose it to the outside world using a NodePort Service. You will learn the difference between ClusterIP and NodePort, how to write Service manifests, and how to verify external connectivity — all foundational skills for working with Kubernetes networking.

----------------------------------------------------------------
Create a Namespace for the Lab
Good Kubernetes hygiene starts with isolating workloads in their own namespace. Create a namespace called 
webdemo
✓
 that all subsequent resources will live in.
1. Run 
kubectl create namespace webdemo
✓
 to create the namespace.
2. Confirm it exists with 
kubectl get namespace webdemo
✓
.
You should see the namespace listed with a STATUS of 
Active
✓
.

----------------------------------------------------------------
Deploy the Web Application
Deploy a simple nginx web server into the 
webdemo
✓
 namespace. You will create a Deployment with 2 replicas so you can later observe traffic being load-balanced across pods.
1. Save the following manifest as 
web-deployment.yaml
✓
 inside the toolbox container:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
  namespace: webdemo
  labels:
    app: web
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: nginx
          image: nginx:1.25-alpine
          ports:
            - containerPort: 80
Copy
2. Apply it with 
kubectl apply -f web-deployment.yaml
✓
.
3. Wait for both replicas to be ready: 
kubectl rollout status deployment/web -n webdemo
✓
.
4. Confirm the pods are Running: 
kubectl get pods -n webdemo
✓
.
----------------------------------------------------------------

----------------------------------------------------------------
Expose the Deployment with a NodePort Service
A NodePort Service allocates a port on every cluster node (in the range 30000–32767) and forwards traffic from that port to the matching pods. This makes the application reachable from outside the cluster without a cloud load balancer.
1. Save the following manifest as 
web-service.yaml
✓
:
apiVersion: v1
kind: Service
metadata:
  name: web-nodeport
  namespace: webdemo
spec:
  type: NodePort
  selector:
    app: web
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30080
Copy
2. Apply it with 
kubectl apply -f web-service.yaml
✓
.
3. Inspect the Service: 
kubectl get service web-nodeport -n webdemo
✓
.
Notice the 
PORT(S)
✓
 column shows 
80:30080/TCP
✓
 — port 80 inside the cluster maps to port 30080 on the node.
**Key concepts:**
- 
port
✓
 — the port the Service listens on inside the cluster (ClusterIP)
- 
targetPort
✓
 — the port the container actually listens on
- 
nodePort
✓
 — the port exposed on every node's IP address (external access)

kubectl get endpoints web-nodeport -n webdemo
----------------------------------------------------------------
Verify the Service Endpoints are Healthy
Before testing external connectivity, confirm that Kubernetes has correctly wired the Service to the running pods by checking its Endpoints object.
1. Run 
kubectl get endpoints web-nodeport -n webdemo
✓
 and confirm you see **2 IP:port** entries (one per pod replica).
2. Describe the Service for a full picture: 
kubectl describe service web-nodeport -n webdemo
✓
.
3. Look at the 
Endpoints:
✓
 field in the describe output — it should list two pod IPs on port 80.
If the Endpoints list is empty (
<none>
✓
), the selector does not match any pod labels. Debug with 
kubectl get pods -n webdemo --show-labels
✓
 and compare against the Service 
selector
✓
.
Hide hint
Run 
kubectl get pods -n webdemo --show-labels
✓
 to see the labels on your pods. The label 
app=web
✓
 must be present. If you used the imperative 
kubectl create deployment
✓
 command earlier without labels, patch the deployment with 
kubectl patch deployment web -n webdemo -p '{"spec":{"template":{"metadata":{"labels":{"app":"web"}}}}}'
✓
copy.
----------------------------------------------------------------
Test External Access via NodePort
With the NodePort Service in place, verify that the application is reachable through the node's IP address on port 30080.
**Step 1 — Find the node's internal IP:**
kubectl get nodes -o wide
✓
Note the 
INTERNAL-IP
✓
 of your kind node (e.g. 
172.18.0.2
✓
).
**Step 2 — Curl the NodePort:**
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
curl -s http://${NODE_IP}:30080
Copy
You should see the default **nginx Welcome page** HTML in the response.
**Step 3 — Confirm load balancing across pods:**
for i in $(seq 1 6); do
  curl -s http://${NODE_IP}:30080 | grep -o '<title>.*</title>'
done
Copy
All 6 requests should return the nginx welcome title, proving the Service is routing traffic correctly to both replicas.
**Bonus:** Run 
kubectl logs -n webdemo -l app=web
✓
 and observe access log entries appearing on both pods.