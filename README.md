![DevOps Best Practices for Business Strategy Success](https://github.com/user-attachments/assets/1ec9a455-e946-4ccc-ab4f-5e59c46b918e)


#  **DevOpsSphere-finalProject**
 

This project implements a robust three-tier application architecture deployed on a Kubernetes cluster. The setup consists of a frontend, backend, and database, each managed as separate services to ensure modularity, scalability, and resilience. The architecture also integrates modern DevOps practices such as CI/CD, secure secrets management, and comprehensive monitoring.



## **the-three-tier** 

 - Frontend: A user-facing interface built with a modern framework (React) to interact with end-users. This service is deployed with three replicas for high availability.

 - Backend: A RESTful API developed using Flask. It handles business logic and communicates with the database layer. This service also runs with three replicas for load balancing.

 - Database: A single MySQL instance serving as the data storage layer.
 
 
### 1. **Environment Setup**
- Set up a local Kubernetes cluster using **KinD**.
- Install necessary Kubernetes tools like `kubectl`, `Helm`, and others.
- Fork the source code to your own GitHub or GitLab repository for better control and integration in CI/CD.

----

### 1: Set Up KinD Cluster

1. **Install KinD**: Follow the installation instructions in the [KinD documentation](https://kind.sigs.k8s.io/docs/user/quick-start/#installation).
2. **Create a KinD Cluster**: Use the following command to create a KinD cluster with a specific name (e.g., `cluster`).
   ```bash
   kind create cluster --name cluster
   
---
### 2: Docker

- **Build Docker Image**: Build an image from the Dockerfile using:
   ```bash
   docker build -t <your-dockerhub-username>/3-tier-app:tag .
   
- Push Docker Image: Push this image to a Docker registry (e.g., Docker Hub).
    ```bash
    docker push <your-dockerhub-username>/3-tier-app:tag.
- Run Docker Container: Run a container using the newly built image to verify functionality.
     ```bash
     docker run -p <port>:<port> <your-dockerhub-username>/3-tier-app:tag.
- Verify Container: Ensure the application is accessible from the container.     

---
## k8s


### deployments

  - Deployment YAML: is a resource object that provides declarative updates to applications. It manages the lifecycle of Pods (the smallest deployable units in Kubernetes), ensuring that the desired number of replicas are running at any time
  
  - Apply the Deployment to the KinD cluster.
  
     ```bash
     kubectl apply -f <deployment_name>.yaml
  
  - Verify Deployment and Pods: Confirm the deployment and pods are running.
     ```bash
     kubectl get deployments
     kubectl get pods

---

### ingress  

  - install NGINX Ingress Controller: Set up NGINX Ingress to manage external access to services in KinD.

     ```bash
     kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml

  - Create and Apply Ingress YAML: Write an ingress.yaml file to configure access to the Service and apply it.

     ```bash
     kubectl apply -f <ingress_name>.yaml

  - Access Application Externally: Add an entry in /etc/hosts pointing todos.local to your KinD node’s IP (use kubectl get nodes -o wide to find the IP).
  
---

### ConfigMap and Secret

  -   ConfigMap YAML: Add configmap.yaml with necessary configuration data for your application.

  -   Secret YAML: Add secret.yaml with sensitive information (e.g., database credentials).
  
  - Apply ConfigMap and Secret to the cluster.
    ```bash
    kubectl apply -f <configmap_name>.yaml
    kubectl apply -f <secret_name>.yaml
  
  - **Reference ConfigMap and Secret in Pod Configuration**: Update deployment.yaml to use these values.
 
---

###  PersistentVolume and PersistentVolumeClaim

  -  PersistentVolume YAML: Define storage capacity and access in pv.yaml.
  
  -  PersistentVolumeClaim YAML: Request the required storage in pvc.yaml.
  
  - Apply PV and PVC to the cluster.
    
    ```bash
    kubectl apply -f <pv.yml>
    kubectl apply -f <pvc.yml>
  
  - Mount PVC in Deployment Configuration: Update deployment.yaml to mount the PersistentVolumeClaim for data persistence.
  
  
---

## Securing Secrets with Sealed Secrets

**Handling sensitive data such as database credentials and API keys is critical in any deployment. For this project, Sealed Secrets were implemented to securely manage and encrypt secrets within the Kubernetes environment.**

 - install SealedSecret controller using helm
   ```bash
   helm repo add bitnami https://charts.bitnami.com/bitnami
   helm repo update
   helm install sealed-secrets bitnami/sealed-secrets --namespace kube-system
  
 - Verify Installation 
   ```bash
   kubectl get po -n kube-system
   
 - Seal the Secrets
   ```bash
   kubeseal --cert=sealed-secrets-public-key.pem --format yaml < sql-secret.yml > sql-sealedSecret.yml
   kubeseal --cert=sealed-secrets-public-key.pem --format yaml < pullingSecret.yml > pullingSealedSecret.yml
   
 - then apply the secrets and restrt the deployments  

  
  
---

## monitoring && logging

- Install Prometheus and Grafana: Use Helm to deploy monitoring tools:
    ```bash
    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    helm install prometheus prometheus-community/prometheus
    helm install grafana grafana/grafana

- Once Prometheus is set up and collecting metrics, follow these steps to integrate it with Grafana for visualization:

   1. Access Grafana Dashboard
        Open Grafana in your browser using the provided URL or IP address.

   2. Add Prometheus as a Data Source
        - Navigate to Configuration > Data Sources.
        - Click Add data source.
        - Select Prometheus from the list of available data sources.
        - In the HTTP URL field, enter the Prometheus server's URL (e.g., http://prometheus-server:9090 if you're using a service name in Kubernetes).
        - Click Save & Test to ensure Grafana can connect to Prometheus.

   3. Create Dashboards
        - Once the data source is configured, you can start creating dashboards.
        - Use pre-built dashboards from Grafana’s dashboard repository or design custom panels to visualize metrics like CPU usage, memory consumption, request rates, and error rates.

   4. Customize Alerts
        - Set up alerts within Grafana to notify you when certain thresholds are met, improving proactive monitoring.

**This integration provides powerful visualization capabilities, allowing you to gain actionable insights into your application’s performance and health.**


---

## Deploying Jenkins using Ansible on a KinD Cluster


**Jenkins, a leading CI/CD automation server, is essential for managing the build, and deployment pipelines. For this project, we automated the Jenkins deployment using Ansible. The playbook handles the creation and application of various Kubernetes manifests, ensuring a seamless deployment process.**

#### Kubernetes Resources Managed by Ansible


The Ansible playbook is designed to deploy the following Kubernetes resources:

    - Service Account (SA): Provides a unique identity for Jenkins pods.
    - Service (SVC): Exposes Jenkins internally within the cluster.
    - Deployment: Manages the Jenkins pod replicas and ensures high availability.
    - PersistentVolume (PV) and PersistentVolumeClaim (PVC): Provides persistent storage for Jenkins data, ensuring logs, jobs, and configurations are retained.
    - ClusterRole and ClusterRoleBinding: Grants Jenkins the necessary permissions to interact with the Kubernetes API.
    
    
    
#### Playbook Execution

   1. Context Setup: The playbook contains references to the manifests for all the above resources. The Kubernetes YAML files are organized under a specific directory, ensuring structured and manageable deployment.

   2. Automated Deployment: Running the playbook applies all Kubernetes manifests to the KinD cluster:
      ```bash
      ansible-playbook playbook.yml
      
 **This command automates the following:**

   - Creation of necessary Kubernetes resources.
   - Application of configurations.
   - Verification that Jenkins is running and accessible.  

  3. Result: Once the playbook completes, Jenkins is fully operational on the KinD cluster. The web interface is available via the configured Ingress route, providing access to its pipeline management features.
  
  
  - Benefits of Using Ansible is Consistency,Scalability,and Efficiency

---

### Integrating Jenkins Pipeline with Git Repository and Setting Up a Webhook

**Once Jenkins is deployed, you can automate your CI/CD processes by integrating it with your Git repository and setting up a webhook for automatic builds on code changes**
  



-----------------


  





   
   
