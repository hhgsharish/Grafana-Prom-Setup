# Grafana-Prom-Setup

**Setting Up Grafana and Prometheus for Kubernetes Monitoring on AWS with NodePort**

I'll guide you through setting up Prometheus and Grafana using NodePort instead of LoadBalancer for your AWS Kubernetes cluster. 
This is a good choice when you want to avoid the additional cost of an AWS load balancer.

**Prerequisites**

An AWS Kubernetes cluster with 1 master and 2 worker nodes (T2.Medium)
kubectl configured to access your cluster
Helm package manager installed (we'll install this if needed)

**Step 1: Install Helm (if not already installed)**

    curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
    
This command downloads and installs Helm 3, which is a package manager for Kubernetes that simplifies deploying applications.

**Step 2: Add the Prometheus Community Helm Repository**

    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    helm repo update
    
This adds the official Prometheus community repository to Helm and updates the repository information, giving us access to the Prometheus and related charts.

**Step 3: Create a Namespace for Monitoring**

    kubectl create namespace monitoring
    
Creating a dedicated namespace helps organize resources and makes managing permissions easier.

**Step 4: Install Prometheus using Helm with NodePort**


    helm install prometheus prometheus-community/kube-prometheus-stack \
      --namespace monitoring \
      --set prometheus.service.type=NodePort \
      --set prometheus.service.nodePort=30090 \
      --set grafana.service.type=NodePort \
      --set grafana.service.nodePort=30080 \
      --set grafana.adminPassword=admin123 \
      --set alertmanager.service.type=NodePort \
      --set alertmanager.service.nodePort=30093 \
      --version 52.1.0

    
**This command**

Installs the kube-prometheus-stack
Places all components in the monitoring namespace
Sets Prometheus, Grafana, and AlertManager to use NodePort service type
Assigns specific node ports for easy access:

Prometheus: 30090
Grafana: 30080
AlertManager: 30093


Sets a default admin password for Grafana
Uses a recent stable version 52.1.0

**Step 5: Verify the Installation**

    kubectl get pods -n monitoring
    
This checks that all the pods are running properly. You should see pods for Prometheus, Grafana, and related components.

**Step 6: Check the Services**

    kubectl get svc -n monitoring
    
Look for the services with type NodePort. You should see the ports we configured for each service.

**Step 7: Access Grafana Dashboard**
To access Grafana, you'll need the public IP or DNS name of any of your Kubernetes nodes and the NodePort we configured (30080).

    Access URL: http://<NODE-PUBLIC-IP>:30080

Username: admin
Password: admin123 (the one you set during installation)

You can get the public IP of your AWS nodes from the EC2 console or by running:

    kubectl get nodes -o wide
    
**Step 8: Access Prometheus and AlertManager (if needed)**
Similarly, you can access:

    Prometheus: http://<NODE-PUBLIC-IP>:30090
    AlertManager: http://<NODE-PUBLIC-IP>:30093

Understanding What We've Deployed

Prometheus: The monitoring system and time series database that collects metrics

Grafana: The visualization tool that creates dashboards from the data Prometheus collects

AlertManager: Handles alerts from Prometheus

Node Exporter: Collects hardware and OS metrics from your Kubernetes nodes

kube-state-metrics: Generates metrics about the state of Kubernetes objects

Security Considerations
Since we're using NodePort, the services are exposed on all nodes in your cluster:

Ensure your AWS security groups allow traffic to the specified node ports (30080, 30090, 30093)

For production environments, consider setting up an ingress controller with TLS or using a reverse proxy

Change the default Grafana password immediately after first login

Additional Notes

Make sure the NodePort range in your Kubernetes cluster includes the ports we've specified (default range is 30000-32767)
