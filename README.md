# Kubernetes Deployment on Minikube

## Leverage Kubernetes Deployments and Services to efficiently deploy, manage, and orchestrate your containerized applications

### Technical Requirements:
- **Containerization:** Docker
- **Container Orchestration:** Kubernetes (Minikube)
- **Package Management:** Helm

## Step 1: Refresh System Packages
Ensure you have the latest package versions and dependencies by updating your system's package lists:
```sh
sudo apt update
```

## Step 2: Set Up Docker Environment
Install Docker to enable local Kubernetes cluster deployment:
```sh
sudo apt install -y docker.io
```

Configure Docker Permissions:
Add your current user to the Docker group to run Docker commands without root privileges:
```sh
sudo usermod -aG docker $USER && newgrp docker
```

## Step 3: Install Minikube
Retrieve the Minikube binary using curl:
```sh
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
```

Make the binary executable and add it to your system's PATH:
```sh
chmod +x minikube
sudo mv minikube /usr/local/bin/
```

## Step 4: Install kubectl
Download the Kubernetes command-line tool, kubectl:
```sh
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

Make it executable and move it into your path:
```sh
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

## Step 5: Launch Minikube
Initialize a single-node Kubernetes cluster within a Docker container:
```sh
minikube start --driver=docker --vm=true
```

Verify the cluster's status:
```sh
minikube status
```

## Step 6: Define Your Kubernetes Deployment
Create a YAML file (deployment.yaml) that outlines your deployment configuration:
```sh
vim deployment.yaml
```

Insert your deployment configuration, replacing `<your-docker-image-name>` with the name of your Docker image:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: <your-docker-image-name>
        ports:
        - containerPort: 5000
```

Apply the deployment configuration:
```sh
kubectl apply -f deployment.yaml
```

## Step 7: Expose Your Application with a Kubernetes Service
Define a service configuration in a YAML file (service.yaml):
```sh
vim service.yaml
```

Specify the service details, including the selector and port mappings:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000
  type: LoadBalancer
```

Apply the service configuration:
```sh
kubectl apply -f service.yaml
```

## Step 8: Access the Application
Open a web browser and navigate to:
```sh
http://<public-ip>:5000/
```

## Using Helm
### Installing Helm on Your Local Machine
Download and install Helm:
```sh
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

### Creating a Helm Chart for Your Project
To create a new Helm chart:
```sh
mkdir node-js
cd node-js
```

Initialize a new Helm chart:
```sh
helm create nodejs
```

### Configure the values.yaml File
Update the `values.yaml` file:
```yaml
replicaCount: 1

image:
  repository: amankumar19/nodejs-web-app:latest

service:
  type: LoadBalancer
  port: 5000
  targetPort: 5000
```

### Update the Service Template (templates/service.yaml)
Modify the service template:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "webapp.fullname" . }}
  labels:
    {{- include "webapp.labels" . | nindent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: {{ .Values.service.targetPort }}
      protocol: TCP
      name: http
  selector:
    {{- include "webapp.selectorLabels" . | nindent 4 }}
```

### Install and Verify the Helm Chart
Deploy the Helm chart:
```sh
helm install nodejs ./nodejs
```

Verify the deployment:
```sh
kubectl get pods -n nodejs
kubectl get svc -n nodejs
```

### Access the Application Locally
To access the application locally:
```sh
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=0
kubectl port-forward service/webapp -n webapp 5000:5000 --address=0.0.0.0 &
```

Open your browser and go to:
```sh
http://localhost:5000
```

You should now be able to access your deployed application locally.

