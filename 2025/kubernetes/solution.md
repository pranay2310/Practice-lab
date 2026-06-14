- Include your Pod YAML

apiVersion: v1ds
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80


    
=======
- Include your Pod YAML

apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80

--------------------------------------------------------

    Create a Dedicated Namespace
Namespaces let you group and isolate Kubernetes resources. Create a namespace called 'first-pod' that will hold everything you build in this lab.
1. Open a shell inside your toolbox container.
2. Run the command to create the namespace.
3. Verify the namespace exists and is Active before moving on.

--------------------------------------------------------


Deploy Your First Pod
Now deploy a Pod named 'my-nginx' in the 'first-pod' namespace using the official nginx image (nginx:1.25). You will write a simple Pod manifest and apply it with kubectl.
1. Create a file called pod.yaml with the following content:
apiVersion: v1
kind: Pod
metadata:
  name: my-nginx
  namespace: first-pod
  labels:
    app: my-nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.25
    ports:
    - containerPort: 80
Copy
2. Apply the manifest: 
kubectl apply -f pod.yaml
✓

3. Watch the Pod status until it shows Running: 
kubectl get pod my-nginx -n first-pod -w
✓

(press Ctrl+C once it shows Running)

--------------------------------------------------------

TASK 3 OF 5
Done
20 pts
Inspect the Pod with kubectl describe
A running Pod holds a wealth of information. Use 'kubectl describe' to read its full details, then answer the following by examining the output:
1. Run: 
kubectl describe pod my-nginx -n first-pod
✓

2. Find and note: the Node the Pod was scheduled on, the Pod IP address, the container image, and the Events section at the bottom.
3. Also retrieve just the Pod IP using a jsonpath query:
kubectl get pod my-nginx -n first-pod -o jsonpath='{.status.podIP}'
✓
The Events section is especially important — it shows exactly what Kubernetes did to start your Pod (Scheduled → Pulling → Pulled → Created → Started).

--------------------------------------------------------

TASK 4 OF 5
Done
20 pts
Read the Pod Logs
Container logs are your first debugging tool. Retrieve the nginx access and startup logs from your running Pod.
1. Fetch all current logs:
kubectl logs my-nginx -n first-pod
✓
2. Stream logs live (press Ctrl+C to stop):
kubectl logs my-nginx -n first-pod -f
✓
3. Show only the last 10 lines:
kubectl logs my-nginx -n first-pod --tail=10
✓
You should see nginx startup messages confirming the server is ready. Note the '--tail' and '-f' flags — you will use them constantly when debugging real applications.

--------------------------------------------------------
TASK 5 OF 5
Done
20 pts
Execute a Command Inside the Running Pod
Just like 'docker exec', kubectl lets you run commands inside a running container. Use this to verify nginx is serving content and to explore the container filesystem.
1. Run a one-shot command inside the Pod to check the nginx welcome page:
kubectl exec my-nginx -n first-pod -- curl -s http://localhost:80
✓
2. Check the nginx version:
kubectl exec my-nginx -n first-pod -- nginx -v
✓
3. Open an interactive shell (type 'exit' when done):
kubectl exec -it my-nginx -n first-pod -- /bin/bash
✓
4. Inside the shell, run:
ls /usr/share/nginx/html
cat /usr/share/nginx/html/index.html
exit
This confirms the Pod is fully operational and you can interact with it just like a local process.
--------------------------------------------------------
