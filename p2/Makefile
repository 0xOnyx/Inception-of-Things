# Makefile for installing, configuring, and verifying k3s & Kubernetes applications

.PHONY: uninstall install install_kubectl check cluster_info contexts clear-cluster namespace configmaps deploy services ingress verify nodes hosts curl-test all

# Uninstall k3s and kubectl
uninstall:
	@echo "Uninstalling k3s and kubectl..."
	sudo /usr/local/bin/k3s-uninstall.sh
	sudo rm -f $$(which kubectl)
	@echo "Removing IP address from eth0 interface..."
	sudo ip addr del 192.168.56.110/24 dev eth0

# Main installation target
install: install_kubectl
	@echo "Installing k3s..."
	curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--node-ip=192.168.56.110" sh -
	@echo "Downloading kubectl..."
	curl -LO "https://dl.k8s.io/release/$$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
	sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
	@echo "Setting permissions and environment variable for kubeconfig..."
	sudo chmod 644 /etc/rancher/k3s/k3s.yaml
	export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
	@echo "Adding IP address to eth0 interface..."
	sudo ip addr add 192.168.56.110/24 dev eth0

# Check k3s node status
check:
	@echo "Checking k3s nodes..."
	kubectl get nodes

# Display Kubernetes API server info
cluster_info:
	@echo "Fetching cluster info..."
	kubectl cluster-info

# Display kubectl contexts
contexts:
	@echo "Getting kubectl contexts..."
	kubectl config get-contexts

# Clear the cluster (all resources, configmaps, ingress, and namespace)
clear-cluster:
	@echo "Clearing the cluster..."
	kubectl delete all --all -A
	kubectl delete configmap --all -A
	kubectl delete ingress --all -A
	kubectl delete namespace apps

# Create namespace
namespace:
	@echo "Creating namespace..."
	kubectl apply -f namespace-app.yaml

# Apply ConfigMaps
configmaps:
	@echo "Applying ConfigMaps..."
	kubectl apply -f configmap/configmap-app1.yaml
	kubectl apply -f configmap/configmap-app2.yaml
	kubectl apply -f configmap/configmap-app3.yaml

# Deploy applications
deploy:
	@echo "Deploying applications..."
	kubectl apply -f deployment/deployment-app1.yaml
	kubectl apply -f deployment/deployment-app2.yaml
	kubectl apply -f deployment/deployment-app3.yaml

# Create services
services:
	@echo "Creating services..."
	kubectl apply -f service/service-app1.yaml
	kubectl apply -f service/service-app2.yaml
	kubectl apply -f service/service-app3.yaml

# Configure ingress
ingress:
	@echo "Configuring ingress..."
	kubectl apply -f ingress/ingress-app.yaml

# Verify the setup
verify:
	@echo "Verifying setup in namespace 'apps'..."
	kubectl get all -n apps
	@echo "Verifying ingress..."
	kubectl get ingress -n apps

# Show node details
nodes:
	@echo "Showing nodes with detailed info..."
	kubectl get nodes -o wide

# Modify /etc/hosts to add application domains
hosts:
	@echo "Adding entries to /etc/hosts..."
	echo -e "\n192.168.56.110 app1.com\n192.168.56.110 app2.com\n192.168.56.110 app3.com" | sudo tee -a /etc/hosts

wait:
	@echo "Waiting for 30 seconds..."
	sleep 30

# Test connectivity via curl
curl-test:
	@echo "Testing connectivity with curl..."
	curl http://192.168.56.110
	curl http://app1.com 
	curl http://app2.com
	curl http://app3.com

# All: perform full installation and deployment
all: install namespace configmaps deploy services ingress verify nodes hosts wait curl-test
