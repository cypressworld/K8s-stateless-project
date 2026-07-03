****************************
only Notepad and Command Prompt (cmd) on your local system to practice everything! You do not need to install heavy IDEs or make complex configuration updates to learn this production workflow.

Since your system currently recognizes gh (GitHub CLI) but doesn't recognize standard git, we will structure this approach so you create the files in Notepad, use cmd to easily push them to GitHub with gh, and look at the real execution outputs in your browser and Killercoda.

Here is the exact, step-by-step practical lab you can follow right now.

🏛️ The Directory Structure on Your PC
We will create a clean workspace folder structure right on your desktop:

Plaintext
C:\Users\USER\Desktop\k8s-stateless-project\
 ├── .github\
 │    └── workflows\
 │         └── deploy-check.yml
 └── core-manifests\
      ├── 01-deployment.yaml
      └── 02-service.yaml
🛠️ Step-by-Step Hands-on Implementation
Phase A: Building the Directories Using cmd
Press the Windows Key, type cmd, and press Enter to open your Command Prompt.

Run these commands line-by-line to build your isolated folder layout on your desktop:

DOS
cd C:\Users\USER\Desktop
mkdir k8s-stateless-project
cd k8s-stateless-project
mkdir core-manifests
mkdir .github
mkdir .github\workflows
Phase B: Creating the Files Using Notepad
Now, let's use Notepad to write our code files and save them directly inside those directories.

1. Create the Deployment Manifest:

In your Command Prompt, type this to open a blank file in Notepad:

DOS
notepad core-manifests\01-deployment.yaml
Notepad will ask: "Cannot find the core-manifests\01-deployment.yaml file. Do you want to create a new file?" Click Yes.

Paste the following textbook Kubernetes code into Notepad, then press Ctrl + S to save it and close Notepad:

YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment-readiness
  labels:
    app: nginx
    environment: test
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
      environment: test
  template:
    metadata:
      labels:
        app: nginx
        environment: test
    spec:
      containers:
      - name: nginx
        image: nginx:1.25.4
        ports:
        - containerPort: 80
        command:
        - /bin/sh
        - -c
        - |
          echo "Served by Pod with IP address: $(hostname -i)" > /usr/share/nginx/html/index.html
          nginx -g "daemon off;"
        readinessProbe:
          httpGet:
            path: /ready
            port: 80
          initialDelaySeconds: 5
          timeoutSeconds: 10
          periodSeconds: 2
          successThreshold: 1
          failureThreshold: 2
2. Create the Service Manifest:

In your Command Prompt, type:

DOS
notepad core-manifests\02-service.yaml
Click Yes to create the file, paste this code, save (Ctrl + S), and close it:

YAML
apiVersion: v1
kind: Service
metadata:
  name: nginx-service-example
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
    nodePort: 32563
  selector:
    app: nginx
    environment: test
3. Create the GitHub Actions Validation Workflow:

In your Command Prompt, type:

DOS
notepad .github\workflows\deploy-check.yml
Click Yes to create the file, paste this automated validation code, save (Ctrl + S), and close it:

YAML
name: K8s Blueprint Lint Validation
on: [push, pull_request]

jobs:
  validate-syntax:
    runs-on: ubuntu-latest
    steps:
      - name: Pull Code Base From GitHub
        uses: actions/checkout@v4

      - name: Validate Deployment YAML Syntax Integrity
        run: |
          python3 -c "import yaml; yaml.safe_load(open('core-manifests/01-deployment.yaml'))"
          python3 -c "import yaml; yaml.safe_load(open('core-manifests/02-service.yaml'))"
          echo "✅ YAML manifest formatting checks passed successfully!"
Phase C: Initializing and Pushing via cmd using gh
Since you already have gh version 2.89.0 configured on your system, we can bypass the local git path error entirely by initializing the repo and authenticating directly through the GitHub CLI tool.

Run these tracking and publishing commands inside your Command Prompt:

DOS
# 1. Log into your cloud account via cmd (if not logged in already)
gh auth login



******************
🛠️ The Complete Supplement Guide: Filling the Gaps

1. Practice Creating a Skeleton Template (The Declarative Way)
Before manually writing a YAML file from scratch, cloud engineers generate a starter file using the --dry-run flag. Run this to see how Kubernetes builds the layout automatically:

Bash
kubectl create deployment my-deployment --replicas=1 --image=nginx:latest --dry-run=client --port=80 -o yaml
2. Test In-Cluster Network Routing via k8sutils
Your textbook notes try to download an external helper pod from a relative directory path (../Chapter07/k8sutils.yaml). Since that file path doesn't exist inside your isolated browser workspace, run this command to spin up a native, live network diagnostic pod instantly:

Bash
# 1. Spin up an interactive utilities tester pod natively
kubectl run k8sutils --image=curlimages/curl --sleep 3600

# 2. Wait until the test tool is active
kubectl wait --for=condition=Ready pod/k8sutils --timeout=30s

# 3. Test internal DNS cluster routing directly to your service
kubectl exec -it k8sutils -- curl http://nginx-service-example.default.svc.cluster.local
Expected Output: It will print the raw HTML data containing <title>Welcome to nginx!</title>, proving your microservices are successfully communicating over internal overlay networks.

3. Practice Imperative Exposing
Instead of writing a service YAML file by hand, you can command the cluster master node to generate a network endpoint directly from a running deployment:

Bash
kubectl expose deployment nginx-deployment-readiness --name=nginx-exposed-service --type=LoadBalancer --port=80

4. Build a Live Canary Deployment Setup
To demonstrate how a Canary Release functions under real load-balanced conditions, apply both stable and canary configurations simultaneously inside your sandbox:

Bash
# 1. Create a combined manifest file matching your appendix notes
cat << 'EOF' > canary-lab.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-stable
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      tier: frontend
  template:
    metadata:
      labels:
        app: myapp
        tier: frontend
        version: stable
    spec:
      containers:
      - name: web-app
        image: nginx:1.24
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-canary
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
      tier: frontend
  template:
    metadata:
      labels:
        app: myapp
        tier: frontend
        version: canary
    spec:
      containers:
      - name: web-app
        image: nginx:1.25
---
apiVersion: v1
kind: Service
metadata:
  name: canary-routing-service
spec:
  ports:
  - port: 80
  selector:
    app: myapp
    tier: frontend
EOF

# 2. Deploy the architecture layout
kubectl apply -f canary-lab.yaml

# 3. Check how the service distributes traffic across versions
kubectl describe svc canary-routing-service
Notice that the Service targeting selector only points to app: myapp, tier: frontend. Because it omits the version label, it will automatically load-balance incoming client traffic across all 4 pods—routing roughly 75% of users to the Stable release and 25% to the Canary release safely!

*********************************

