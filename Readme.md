# NOTE: Ensure that your EKS cluster is up and running before proceeding (Bastion host as well)

🔧 1. Install kubectl in Bastion host
# official binary – recommended
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Make it executable:
chmod +x kubectl

# Move it to PATH:
sudo mv kubectl /usr/local/bin/

# Verify:
kubectl version --client

🔧 2. Install Git
sudo yum install git -y

🔧 3. Install kubectx (and kubens)
git clone https://github.com/ahmetb/kubectx.git ~/.kubectx

# Add to PATH:
sudo ln -s ~/.kubectx/kubectx /usr/local/bin/kubectx
sudo ln -s ~/.kubectx/kubens /usr/local/bin/kubens

# Verify:
kubectx (It will not show anything as it is not conected to eks yet)
kubens  (It will not show anything as it is not conected to eks yet)


🧪 4. Configure Access (Important)
Create an "Access key" for your user in AWS
Go to IAM -> Users -> SELECT_YOUR_USER  -> Security credentials -> Access keys -> Create access key

# Now in Ec2, configure: AWS Account setup
Aws configure

# Provide:
AWS Access Key ID
AWS Secret Access Key
Region

# Add EKS cluster to Kubectx
aws eks update-kubeconfig --region <region> --name <cluster-name>

# Install Helm (official script)
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

✅ Expose Argo CD with LoadBalancer

🚀 Step 1: Create the namespace manually
kubectl create namespace argocd

🚀 Step 2: Install Argo CD (you already did / will do)
kubectl create -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml


# Note: You need to patch the service after installation. So that can access the url via internet
🚀 Step 3: Change service to LoadBalancer
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

⏳ Step 4: Wait for External IP
kubectl get svc argocd-server -n argocd

# You’ll see something like:
NAME             TYPE           EXTERNAL-IP
argocd-server    LoadBalancer   <pending>

# After a bit:
EXTERNAL-IP: a1b2c3d4***************.elb.amazonaws.com # It takes some time to load into UI

⏳ Step 5: Open the Above URL
# Username for ArgoCD App
admin

# Get password for ArgoCD App
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d

⏳ Step 6:
# Argo CD will handle Helm automatically, To install both Apps

# Create namespaces manually for Nginx and Grafana app
kubectl create namespace pathnex
kubectl create namespace monitoring

# Make sure to present in "ArgoCd_lab" folder
kubectl apply -f argo-apps/      

# Sample of the output, These EXTERNAL-IP will be used to login to argocd, grafana and nginx. (Your values will be different)

kubectl get svc -A
NAMESPACE      NAME                                      TYPE           CLUSTER-IP       EXTERNAL-IP
argocd         argocd-server                             LoadBalancer   172.20.233.130   a15e3a35b978f4cd4bcb080971df360e-523335367.ap-south-1.elb.amazonaws.com    80:31412/TCP,443:32045/TCP   10m                                                               443/TCP                      4h7m
monitoring     pathnex-grafana-service                   LoadBalancer   172.20.95.153    aa32e4bf413644023a0e7630754be889-1022658738.ap-south-1.elb.amazonaws.com   80:31011/TCP                 6m56s
pathnex        pathnex-nginx-service                     LoadBalancer   172.20.156.215   a85c1db8575934cf782e2e0ba1eef0aa-1472485196.ap-south-1.elb.amazonaws.com   80:31425/TCP                 14m

# To force sync after making a change.
kunectl patch application pathnex-nginx -n argocd   --type merge   -p '{"operation":{"sync":{}}}'

# After completion, To delete the namespace and its resources 
kubectl delete namespace monitoring pathnex argocd

# Note: Don't forget to delete the EKS cluster and bastion host after the lab.
