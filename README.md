
<body>
  <h1>Microservices Architecture with Kubernetes, ECR, and CI/CD Pipeline</h1>

  <h2>1. Set up a Kubernetes Cluster:</h2>
  
  <p>Follow the steps below to set up a Kubernetes cluster:</p>
  <ol>
    <li>Install Minikube by following the instructions on the <a href="https://minikube.sigs.k8s.io/docs/start/">official Minikube site</a>.</li>
    <li>Start Minikube:</li>
    <pre><code>minikube start</code></pre>
    <li>Verify Minikube is running:</li>
    <pre><code>kubectl get nodes</code></pre>
  <p>Follow the steps below to set up a Kubernetes cluster:</p>
  <ol>
    <li>Install Kubernetes using your preferred method (e.g., Minikube, kubeadm, etc.).</li>
    <li>Initialize the Kubernetes cluster with the following command:</li>
    <pre><code>kubectl apply -f cluster.yaml</code></pre>
  </ol>

  <h2>2. Docker and ECR Setup:</h2>
  <p>Follow the steps below to set up Docker and configure AWS ECR:</p>
  
  <ol>
    <li>Install Docker by following the instructions on the <a href="https://docs.docker.com/get-docker/">official Docker site</a>.</li>
    <li>Create a Dockerfile for your Flask application:</li>
  <ol>
    <li>Install Docker on your local machine.</li>
    <li>Create a Dockerfile for your Flask application.</li>
    <li>Build the Docker image with the following command:</li>
    <pre><code>docker build -t project-demo .</code></pre>
    <li>Tag the Docker image with your ECR repository URI:</li>
    <pre><code>docker tag project-demo:latest public.ecr.aws/q8b2j4d8/project-demo:latest</code></pre>
    <li>Push the Docker image to AWS ECR:</li>
    <pre><code>docker push public.ecr.aws/q8b2j4d8/project-demo:latest</code></pre>
  </ol>

  <h2>3. Ingress Configuration:</h2>
  <p>Configure Ingress to manage public traffic to your Kubernetes cluster:</p>
  <ol>
    <li>Install the Nginx Ingress Controller:</li>
    <pre><code>kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml</code></pre>
    <li>Verify the Ingress Controller is running:</li>
    <pre><code>kubectl get pods -n ingress-nginx</code></pre>
    <li>Apply your Ingress resource configuration:</li>
    <pre><code>kubectl apply -f ingress.yaml</code></pre>
    <li>Edit your hosts file to map your domain to Minikube's IP:</li>
    
   <h2>4. CI/CD Pipeline with GitHub Actions:</h2>
  <p>Set up a pipeline using GitHub Actions to automate building, pushing, and deploying your application:</p>
  <ol>
    <li>Navigate to your GitHub repository's Settings > Secrets.</li>
    <li>Add the following secrets:</li>
    <ul>
      <li>AWS_ACCOUNT_ID: Your AWS account ID.</li>
      <li>AWS_REGION: The AWS region where your ECR repository is located.</li>
      <li>AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY: AWS IAM credentials with necessary permissions.</li>
    </ul>
    <li>Create a GitHub Actions workflow file (<code>.github/workflows/ci-cd-pipeline.yml</code>) with the following content:</li>
    <pre><code>
name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: windows-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v1

      - name: Log in to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1
        
      - name: Build, tag, and push image to ECR
        env:
          ECR_REGISTRY: \${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.\${{ secrets.AWS_REGION }}.amazonaws.com
          ECR_REPOSITORY: project-demo
          IMAGE_TAG: latest
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG

      - name: Start Minikube
        run: minikube start

      - name: Wait for Minikube to be ready
        run: minikube status

      - name: Configure kubectl to connect to Minikube
        run: kubectl config use-context minikube

      - name: Deploy application to Kubernetes
        run: kubectl apply -f cluster.yaml

      - name: Apply Ingress configuration
        run: kubectl apply -f ingress.yaml
    </code></pre>
  </ol>
 
</body>
</html>
