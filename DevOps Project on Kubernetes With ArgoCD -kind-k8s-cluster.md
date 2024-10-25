To create an EC2 instance of type `t2.medium` with 16 GB storage and a key pair, follow these steps in the AWS Management Console or by using the AWS CLI.

### Using the AWS Management Console

1. **Go to the EC2 Dashboard**:

   - In the AWS Management Console, navigate to **EC2 Dashboard** > **Instances** > **Launch Instances**.

2. **Choose an Amazon Machine Image (AMI)**:

   - Select an appropriate AMI. For a basic Linux instance, choose **Amazon Linux 2 AMI** (64-bit x86).

3. **Choose Instance Type**:

   - Choose `t2.medium` (2 vCPUs, 4 GB RAM) as the instance type, then click **Next: Configure Instance Details**.

4. **Configure Instance Details**:

   - You can leave most settings as default or adjust them according to your network configuration (e.g., VPC, subnet).
   - Ensure that **Auto-assign Public IP** is enabled if you want the instance to be publicly accessible.

5. **Add Storage**:

   - Set the **size** of the root volume to `16 GB`.
   - Leave the **Volume Type** as **General Purpose SSD (gp2)** or select another based on your preference.

6. **Add Tags** (Optional):

   - Add a tag with a `Key` like `"Name"` and a `Value` like `"k8s-cluster-key.pem`"` to help identify this instance.

7. **Configure Security Group**:

   - Select an existing security group or create a new one.
   - For SSH access, add a rule with **Type** as `SSH`, **Protocol** as `TCP`, **Port Range** as `22`, and **Source** as `My IP` for secure SSH access.

8. **Create or Select a Key Pair**:

   - In the **Key Pair** section, select an existing key pair or create a new one if you don't have one.
   - **Download** the key pair file (k8s-cluster-key.pem`) and keep it secure as you’ll need it to SSH into the instance.

9. **Review and Launch**:

   - Review your settings, and click **Launch** to start your instance.

10. **Verify the Instance**:

- Once launched, check the **Instances** section to see your new `t2.medium` instance.

---

Here's a step-by-step documentation based on the provided commands for accessing a remote EC2 instance to install `containerd`:

---

## Step-by-Step Guide for Installing `containerd` on a Remote EC2 Instance

### Prerequisites

- Ensure you have an SSH key file (e.g., `k8s-cluster-key.pem`) with access permissions to connect to the remote EC2 instance.
- Verify that your EC2 instance is set up with the necessary security group rules (allowing SSH on port 22) and is ready to install `containerd`.

---

### Step 1: Navigate to the Downloads Directory

Start by navigating to the `Downloads` directory where your key file (`k8s-cluster-key.pem`) is located:

```bash
cd Downloads
```

### Step 2: Verify the Contents of the Downloads Directory

List the contents of the `Downloads` directory to ensure the key file is there:

```bash
ls
```

Look for the `k8s-cluster-key.pem` file in the output.

### Step 3: Set the Appropriate Permissions on the SSH Key File

Before connecting to the EC2 instance, set permissions on the SSH key file to make it secure. This is necessary for SSH to recognize the file as a private key:

```bash
chmod 400 k8s-cluster-key.pem
```

> **Note**: The `chmod 400` command limits the file’s permissions to read-only for the user, which is required by SSH.

### Step 4: Connect to the EC2 Instance

With the key file permissions set, you can now connect to the remote EC2 instance. Use the following SSH command, replacing the instance URL with your EC2 instance's public DNS:

```bash
ssh -i "k8s-cluster-key.pem" ubuntu@ec2-3-109-182-102.ap-south-1.compute.amazonaws.com
```

This command initiates an SSH session using the key file `k8s-cluster-key.pem`, logging in as the `ubuntu` user on the specified EC2 instance.

### Step 5: Verify the SSH Connection

After executing the command, you should now be logged into the EC2 instance. Confirm you are in the EC2 instance by checking the system information:

```bash
uname -a
```

Once inside, you can begin with the installation steps for `containerd`.

---

### Step 6: Download and Install `containerd` (On the EC2 Instance)

Now that you are connected to the EC2 instance, follow the steps below to install `containerd`.

#### Update Package Lists

```bash
sudo apt-get update
```

#### Install `containerd`

```bash
sudo apt-get install -y containerd
```

#### Confirm the Installation

```bash
containerd --version
```

This should display the installed version of `containerd`, indicating a successful installation.

---

### Step 7: Exit the EC2 Instance

Once the installation is complete, you can log out of the EC2 instance:

```bash
exit
```

---

This completes the process of connecting to your EC2 instance and installing `containerd`.

Here's a detailed, step-by-step guide based on the commands executed in your session. This guide includes the setup for installing Docker, verifying access permissions, and managing Docker commands without requiring root access on an EC2 instance.

---

## Step-by-Step Guide to Install Docker and Configure User Permissions on Ubuntu EC2

### Prerequisites

- SSH access to your Ubuntu EC2 instance, configured with the necessary security group rules for SSH access.

---

### Step 1: Update Package Lists

Begin by updating the package lists to ensure you have the latest information about available packages.

```bash
sudo apt update
```

This command connects to the Ubuntu package repository to fetch the latest metadata, which is necessary before installing new packages.

---

### Step 2: Install Docker and Containerd

Install Docker (which includes `containerd` by default) with the following command:

```bash
sudo apt-get install docker.io -y
```

This command installs `docker.io` and dependencies, including `containerd` and utilities needed for Docker networking and bridge management.

---

### Step 3: Verify Docker Installation

After installation, verify that Docker is running by checking the status of the Docker service:

```bash
sudo systemctl status docker
```

You should see that the Docker service is active and running.

---

### Step 4: Test Docker with a Simple Command

To verify the Docker setup, use the following command:

```bash
docker ps
```

However, if you encounter a **permission denied** error when trying to run Docker commands, it is because Docker requires elevated permissions to connect to the Docker daemon socket (`/var/run/docker.sock`).

---

### Step 5: Add Your User to the Docker Group

To enable Docker access without `sudo`, add your user to the `docker` group:

```bash
sudo usermod -aG docker $USER && newgrp docker
```

This command appends your current user to the `docker` group, allowing Docker commands to be run without root access.

### Step 6: Verify Docker Permissions

Now that your user is in the `docker` group, verify that you can run Docker commands:

```bash
docker ps
```

If you see the list of running containers (or an empty list if there are no containers), Docker is set up correctly.

---

### Summary

You have successfully installed Docker, configured user permissions to avoid the need for `sudo` with Docker commands, and verified the setup on your EC2 instance. You’re now ready to manage containers on your instance.

Here's a step-by-step guide to create and execute a script to install `kind` (Kubernetes in Docker) on your Ubuntu EC2 instance. This includes writing the installation script, granting it executable permissions, and verifying the installation.

---

## Step-by-Step Guide to Install Kind on Ubuntu EC2

### Step 1: Create a Directory for Kubernetes Installation

Create a directory to store the installation script and related files:

```bash
mkdir k8s-install
cd k8s-install/
```

---

### Step 2: Create the Installation Script for Kind

Open a new file named `kind_install.sh` in the `vim` editor:

```bash
vim kind_install.sh
```

Within the `vim` editor, add the following lines to download and install `kind`:

```#!/bin/bash

# For AMD64 / x86_64

[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo cp ./kind /usr/local/bin/kind
rm -rf kind
```

To save and exit in `vim`, press `Esc`, then type `:wq` and press `Enter`.

---

### Step 3: Make the Script Executable

Grant executable permissions to the `kind_install.sh` script:

```bash
chmod +x kind_install.sh
```

---

### Step 4: Run the Kind Installation Script

Execute the installation script:

```bash
./kind_install.sh
```

This script downloads the `kind` binary and moves it to `/usr/local/bin`, making it available in your PATH.

---

### Step 5: Verify the Installation

Check that `kind` was installed successfully by verifying the version:

```bash
kind --version
```

If the installation was successful, you should see output similar to:

```plaintext
kind version 0.20.0
```

---

### Summary

You have successfully installed `kind` on your Ubuntu EC2 instance by creating and running an installation script.

Here's how to create and use a `kind` configuration file to set up a custom Kubernetes cluster on your EC2 instance.

---

## Step-by-Step Guide to Create and Configure a Kind Cluster

### Step 1: Create a Kind Configuration File

In your `k8s-install` directory, create a configuration file named `config.yml` using the `vim` editor:

```bash
vim config.yml
```

Within the `vim` editor, define your custom cluster configuration. Here’s an example configuration file:

```yaml
# config.yml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4

nodes:
  - role: control-plane
    image: kindest/node:v1.30.0
  - role: worker
    image: kindest/node:v1.30.0
  - role: worker
    image: kindest/node:v1.30.0
```

This configuration file specifies:

- A control-plane node and two worker nodes.
- Custom pod and service subnet settings.

To save and exit in `vim`, press `Esc`, type `:wq`, and hit `Enter`.

---

### Step 2: Create a Kind Cluster Using the Configuration

Use the `kind` command to create a cluster based on the configuration file:

```bash
kind create cluster --config=config.yml --name=my-cluster
```

- The `--config=config.yml` flag points to the custom configuration file.
- The `--name=my-cluster` flag gives the cluster a specific name (`my-cluster`).

---

### Step 3: Verify the Cluster Creation

Once the cluster creation process completes, verify your cluster by checking the list of clusters:

```bash
kind get clusters
```

You should see `my-cluster` listed in the output, confirming it was created successfully.

---

### Summary

You have now created a Kubernetes cluster using `kind` with a custom configuration on your Ubuntu EC2 instance. This setup includes specifying the number and types of nodes and networking settings tailored to your requirements.

Here's a guide to creating a script to install `kubectl` on your EC2 instance.

---

## Step-by-Step Guide to Install Kubectl Using a Script

### Step 1: Create the Installation Script

In your `k8s-install` directory, create a new script named `install_kubectl.sh` using `vim`:

```bash
vim install_kubectl.sh
```

In the `vim` editor, add the following commands to download and install `kubectl`:

```bash
#!/bin/bash

# Variables
VERSION="v1.30.0"
URL="https://dl.k8s.io/release/${VERSION}/bin/linux/amd64/kubectl"
INSTALL_DIR="/usr/local/bin"

# Download and install kubectl
curl -LO "$URL"
chmod +x kubectl
sudo mv kubectl $INSTALL_DIR/
kubectl version --client

# Clean up
rm -f kubectl

echo "kubectl installation complete."
```

To save and exit in `vim`, press `Esc`, type `:wq`, and hit `Enter`.

---

### Step 2: Make the Script Executable

After creating the script, make it executable by running:

```bash
chmod +x install_kubectl.sh
```

### Step 3: Run the Installation Script

Execute the script to install `kubectl`:

```bash
./install_kubectl.sh
```

You should see output similar to:

```
Client Version: v1.30.0
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
kubectl installation complete.
```

---

### Step 4: Verify Kubectl Installation

To confirm that `kubectl` is properly installed and accessible, use:

```bash
kubectl version --client
```

If installed successfully, `kubectl` will return its client version.

---

### Summary

You've now created a script to install `kubectl` on your Ubuntu EC2 instance, making it quick to set up and verify the installation. This script can also be reused on similar environments for easy `kubectl` setup.

### Verify Kubernetes Node Status

```bash
kubectl get nodes
```

Ensure all nodes in the cluster are up and running.

### Check Namespaces in the Cluster

```bash
kubectl get ns
```

List namespaces to verify Kubernetes components are active.

### Deploy Argo CD to the Cluster

1. Create the Argo CD namespace:
   ```bash
   kubectl create namespace argocd
   ```
2. Apply the Argo CD installation manifest:
   ```bash
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

This setup covers the essential steps for installing Docker, setting up a `kind` Kubernetes cluster, and deploying Argo CD in a `kind`-based Kubernetes environment on an EC2 instance.

### Step 12: Verify Argo CD Services

Check the services in the `argocd` namespace to ensure that Argo CD components are deployed and running:

```bash
kubectl get svc -n argocd
```

This command lists all services associated with Argo CD in the `argocd` namespace, including `argocd-server`, which is the main service used to access the Argo CD UI.

### Step 13: Expose Argo CD Server with NodePort

To access Argo CD externally, change the `argocd-server` service type to `NodePort`:

```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
```

The `NodePort` setting makes the Argo CD server accessible on a high port on each node, allowing access from outside the cluster.

You can stop it using you can try running the port forwarding command again:

```bash
kubectl port-forward -n argocd service/argocd-server 8443:443 --address=0.0.0.0 &
```

After executing this, you should be able to access the Argo CD server at `https://<public-ip>:8443` or `https://localhost:8443`. Let me know how it goes!

Here's how to edit the inbound rules in your EC2 instance's security group to allow access to the Argo CD server on port `8443`:

### Update EC2 Security Group Inbound Rules

1. **Navigate to the AWS Management Console:**

   - Go to the [AWS Management Console](https://aws.amazon.com/console/).

2. **Select EC2:**

   - In the console, search for and select **EC2**.

3. **Locate Your Instance:**

   - In the left sidebar, click on **Instances**.
   - Find and select the EC2 instance where Argo CD is installed.

4. **Edit Security Group:**

   - In the **Description** tab at the bottom, find the **Security groups** section and click on the security group linked to your instance.

5. **Modify Inbound Rules:**

   - Click on the **Inbound rules** tab.
   - Click on the **Edit inbound rules** button.

6. **Add a New Rule:**

   - Click on **Add rule**.
   - Set the following:
     - **Type:** Custom TCP
     - **Protocol:** TCP
     - **Port range:** 8443
     - **Source:** Anywhere (0.0.0.0/0) (Note: This allows access from any IP address. For better security, consider restricting this to specific IPs.)
     - **Description:** `argocd-server` (optional)

7. **Save Rules:**
   - Click on the **Save rules** button.

### Summary

You have successfully edited the inbound rules of your EC2 instance's security group to allow traffic on port `8443` for the Argo CD server from any IP address. This will enable you to access the Argo CD UI using the URL:

```
https://<public-ip>:8443
```

Replace `<public-ip>` with your instance's public IP address.

If you have any further questions or need additional assistance, feel free to ask!

Retrieve Argo CD admin password

kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
PqoAH0zUqmxBKCuy

Here’s a detailed documentation for setting up Argo CD for the `voting-app`, including all the configurations based on your specifications.

---

# Argo CD Setup Documentation for Voting App

## General Configuration

### Application Name

- **Name:** `voting-app`

### Project Name

- **Name:** `default`

### Sync Policy

- **Policy Type:** `Automatic`
  - This setting ensures that the application automatically syncs its desired state with the live state in the cluster.

### Prune Resources

- **Enabled:** `Yes`
  - When enabled, resources that are no longer defined in the source will be deleted from the cluster.

### Self Heal

- **Enabled:** `Yes`
  - Argo CD will automatically revert any changes made to the application resources outside of Argo CD.

---

## Source Configuration

### Repository URL

- **URL:** `https://github.com/shashidas95/voting-app.git`
  - The Git repository where the application specifications are stored.

### Revision

- **Branch:** `main`
  - The branch from which to sync the application.

### Path

- **Path:** `k8s-specifications`
  - The directory within the repository that contains the Kubernetes resource definitions.

---

## Destination Configuration

### Cluster URL

- **URL:** `https://kubernetes.default.svc`
  - The API server address for the target Kubernetes cluster.

### Namespace

- **Namespace:** `default`
  - The Kubernetes namespace where the application will be deployed.

This documentation outlines the configuration settings for deploying the `voting-app` using Argo CD. Make sure to review each setting for your specific use case and adjust as necessary. If you have any questions or need further modifications, feel free to ask!

Here's a detailed documentation based on the commands you provided for checking the status of your pods and services, as well as setting up port forwarding for your application.

---

# Argo CD Application Deployment and Service Verification Documentation

## Checking Pod Status

### Command

```bash
kubectl get pod
```

### Output

```plaintext
NAME                      READY   STATUS    RESTARTS   AGE
db-597b4ff8d7-2nwr5       1/1     Running   0          5m43s
redis-796dc594bb-8d6n7    1/1     Running   0          5m43s
result-d8c4c69b8-bbr9c    1/1     Running   0          5m43s
vote-69cb46f6fb-d4pks     1/1     Running   0          5m43s
worker-5dd767667f-b2nxj   1/1     Running   0          5m43s
```

### Explanation

- This command retrieves the current status of all pods running in the cluster.
- **READY** indicates the number of containers that are ready vs. the total number of containers in the pod.
- **STATUS** shows whether the pod is running, pending, or has failed.
- **RESTARTS** counts how many times the pod has restarted since its last creation.
- **AGE** indicates how long the pod has been running.

### Pod Status Interpretation

- All listed pods are in the `Running` state, which means they are healthy and operating as expected.

---

## Checking Service Status

### Command

```bash
kubectl get svc
```

### Output

```plaintext
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
db           ClusterIP   10.96.153.99    <none>        5432/TCP         14m
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP          145m
redis        ClusterIP   10.96.102.43    <none>        6379/TCP         14m
result       NodePort    10.96.53.226    <none>        5001:31001/TCP   14m
vote         NodePort    10.96.210.161   <none>        5000:31002/TCP   14m
```

### Explanation

- This command lists all services running in the cluster.
- **NAME** specifies the service name.
- **TYPE** indicates the service type (e.g., ClusterIP, NodePort).
- **CLUSTER-IP** is the internal IP address of the service.
- **EXTERNAL-IP** shows the external access point (if applicable).
- **PORT(S)** lists the ports that the service is exposing.

### Service Status Interpretation

- Both `result` and `vote` services are of type `NodePort`, meaning they are accessible from outside the cluster on the specified ports.
- The `db` and `redis` services are of type `ClusterIP`, meaning they are only accessible from within the cluster.

---

## Setting Up Port Forwarding

### Command to Forward Traffic for Vote Service

```bash
kubectl port-forward svc/vote 5000:5000 --address=0.0.0.0 &
```

### Command to Forward Traffic for Result Service

```bash
kubectl port-forward svc/result 5001:5001 --address=0.0.0.0 &
```

### Explanation

- The `kubectl port-forward` command allows you to access the services running in the cluster locally.
- **svc/vote** and **svc/result** specify the services for which traffic is being forwarded.
- **5000:5000** and **5001:5001** specify the local port (left) and the service port (right).
- **--address=0.0.0.0** makes the service accessible from all network interfaces.

### Port Forwarding Status

- Once the command is executed, the output will indicate that forwarding has begun:

```plaintext
Forwarding from 0.0.0.0:5000 -> 80
```

---

## Conclusion

This documentation outlines the process of checking the status of your Kubernetes pods and services, as well as setting up port forwarding to access your application from outside the cluster. Make sure to verify that the services are reachable via the specified ports after setting up port forwarding. If you have further questions or require assistance with additional configurations, feel free to ask!

---

Feel free to modify any sections to better fit your specific needs!

Here's the updated documentation, including the steps to modify the inbound rules in the EC2 instance security settings for ports 5000 and 5001:

---

# Argo CD Application Deployment and Service Verification Documentation

## Checking Pod Status

### Command

```bash
kubectl get pod
```

### Output

```plaintext
NAME                      READY   STATUS    RESTARTS   AGE
db-597b4ff8d7-2nwr5       1/1     Running   0          5m43s
redis-796dc594bb-8d6n7    1/1     Running   0          5m43s
result-d8c4c69b8-bbr9c    1/1     Running   0          5m43s
vote-69cb46f6fb-d4pks     1/1     Running   0          5m43s
worker-5dd767667f-b2nxj   1/1     Running   0          5m43s
```

### Explanation

- This command retrieves the current status of all pods running in the cluster.
- **READY** indicates the number of containers that are ready vs. the total number of containers in the pod.
- **STATUS** shows whether the pod is running, pending, or has failed.
- **RESTARTS** counts how many times the pod has restarted since its last creation.
- **AGE** indicates how long the pod has been running.

### Pod Status Interpretation

- All listed pods are in the `Running` state, which means they are healthy and operating as expected.

---

## Checking Service Status

### Command

```bash
kubectl get svc
```

### Output

```plaintext
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
db           ClusterIP   10.96.153.99    <none>        5432/TCP         14m
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP          145m
redis        ClusterIP   10.96.102.43    <none>        6379/TCP         14m
result       NodePort    10.96.53.226    <none>        5001:31001/TCP   14m
vote         NodePort    10.96.210.161   <none>        5000:31002/TCP   14m
```

### Explanation

- This command lists all services running in the cluster.
- **NAME** specifies the service name.
- **TYPE** indicates the service type (e.g., ClusterIP, NodePort).
- **CLUSTER-IP** is the internal IP address of the service.
- **EXTERNAL-IP** shows the external access point (if applicable).
- **PORT(S)** lists the ports that the service is exposing.

### Service Status Interpretation

- Both `result` and `vote` services are of type `NodePort`, meaning they are accessible from outside the cluster on the specified ports.
- The `db` and `redis` services are of type `ClusterIP`, meaning they are only accessible from within the cluster.

---

## Setting Up Port Forwarding

### Command to Forward Traffic for Vote Service

```bash
kubectl port-forward svc/vote 5000:5000 --address=0.0.0.0 &
```

### Command to Forward Traffic for Result Service

```bash
kubectl port-forward svc/result 5001:5001 --address=0.0.0.0 &
```

### Explanation

- The `kubectl port-forward` command allows you to access the services running in the cluster locally.
- **svc/vote** and **svc/result** specify the services for which traffic is being forwarded.
- **5000:5000** and **5001:5001** specify the local port (left) and the service port (right).
- **--address=0.0.0.0** makes the service accessible from all network interfaces.

### Port Forwarding Status

- Once the command is executed, the output will indicate that forwarding has begun:

```plaintext
Forwarding from 0.0.0.0:5000 -> 80
```

---

## Updating EC2 Security Group Rules

### Steps to Add Inbound Rules for Ports 5000 and 5001

1. **Login to AWS Management Console:**

   - Go to the [EC2 Dashboard](https://console.aws.amazon.com/ec2/).

2. **Select Security Groups:**

   - In the left navigation pane, click on "Security Groups."

3. **Choose Your Security Group:**

   - Find and select the security group associated with your EC2 instance.

4. **Edit Inbound Rules:**

   - Click on the "Inbound rules" tab, then click on "Edit inbound rules."

5. **Add New Rules:**

   - Click on "Add rule" and configure the following:
     - **Type:** Custom TCP
     - **Protocol:** TCP
     - **Port Range:** 5000
     - **Source:** 0.0.0.0/0 (to allow traffic from anywhere)
     - Click on "Add rule" again and repeat for port 5001.

6. **Save Rules:**
   - Click on "Save rules" to apply your changes.

---

## Conclusion

This documentation outlines the process of checking the status of your Kubernetes pods and services, setting up port forwarding to access your application from outside the cluster, and configuring EC2 security group rules to allow traffic on the specified ports. Ensure that the services are reachable via the specified ports after setting up port forwarding and modifying security rules. If you have further questions or require assistance with additional configurations, feel free to ask!

---

Feel free to make any additional adjustments or let me know if there's anything else you need!

Here’s a comprehensive documentation of the commands and their outputs for deploying the Kubernetes Dashboard, along with an explanation of each step involved:

---

# Kubernetes Dashboard Deployment Documentation

## Overview

This documentation outlines the steps to deploy the Kubernetes Dashboard using `kubectl`. It details the commands executed, their outputs, and subsequent actions taken to set up and access the dashboard.

## 1. Deploy the Kubernetes Dashboard

### Command

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

### Output

```plaintext
namespace/kubernetes-dashboard created
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/dashboard-metrics-scraper created
```

### Explanation

This command deploys the Kubernetes Dashboard along with necessary RBAC roles, bindings, secrets, and other resources.

## 2. Create Namespace (if needed)

### Command

```bash
kubectl create namespace kubernetes-dashboard
```

### Output

```plaintext
namespaces "kubernetes-dashboard" already exists
```

## 3. Apply Custom Dashboard Configuration

### Command

```bash
kubectl apply -f dashboard.yml
```

### Output

```plaintext
serviceaccount/admin-user created
clusterrolebinding.rbac.authorization.k8s.io/admin-user created
```

dashboard.yml

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```

## 6. Port Forwarding to Access the Dashboard

### Command

```bash
kubectl port-forward svc/kubernetes-dashboard -n kubernetes-dashboard 8080:443 --address=0.0.0.0 &
```

### Output

```plaintext
Forwarding from 0.0.0.0:8080 -> 8443
```

### Explanation

This command forwards port 443 of the Kubernetes Dashboard service to port 8080 on your local machine, allowing access to the dashboard through `http://<your-ec2-ip>:8080/`.### Explanation
This command applies any custom configurations defined in `dashboard.yml`, which includes creating a service account and a cluster role binding for the `admin-user`.

## 4. Create an Authentication Token for the Admin User

### Command

```bash
kubectl -n kubernetes-dashboard create token admin-user
```

### Output

```plaintext
eyJhbGciOiJSUzI1NiIsImtpZCI6Ijc4XzFObm1... (token truncated for brevity)
```

### Explanation

This command generates a token for the `admin-user`, which will be used to authenticate when accessing the dashboard.

## Conclusion

After successfully executing the commands above, you should be able to access the Kubernetes Dashboard via your web browser at `http://<your-ec2-ip>:8080/`. Use the authentication token generated earlier for logging in as the admin user.

To delete your Kubernetes cluster, follow these steps:

### 1. Delete Resources in the `kubernetes-dashboard` Namespace

First, delete all resources related to the Kubernetes Dashboard (if you want to clean up before deleting the cluster):

```bash
kubectl delete -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
kubectl delete namespace kubernetes-dashboard
```

### 2. Delete the Cluster Created by `kind`

If you set up your cluster with `kind`, you can delete it with:

```bash
kind delete cluster --name <cluster-name>
```

Replace `<cluster-name>` with the actual name of your cluster (if you didn’t specify a name, the default is `kind`). For example:

```bash
kind delete cluster --name my-cluster
```

### 3. Verify Cluster Deletion

Once the deletion command completes, verify that the cluster no longer exists by listing clusters:

```bash
kind get clusters
```

This should return an empty list if the cluster was deleted successfully.

---

After completing these steps, your cluster and related resources should be fully removed.

link: https://youtu.be/Kbvch_swZWA?si=wVbp2jt-6KeddOcH
