.PHONY: all clean create_cluseter install_gitlab install

all: clean create_cluster install install_gitlab

# Install k3d if not already installed
install_kubectl:
	@echo "Downloading kubectl..."
	curl -LO "https://dl.k8s.io/release/$$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
	sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
	rm kubectl

clean:
	k3d cluster delete p3

create_cluster:
	k3d cluster create p3 -p 8443:443@loadbalancer -p 8888:8888@loadbalancer
	k3d kubeconfig get p3

install_gitlab:
	helm repo add gitlab https://charts.gitlab.io/
	helm repo update
	helm upgrade --install gitlab gitlab/gitlab \
	--namespace gitlab \
	--create-namespace \
	--set global.hosts.domain=localhost \
	--set global.hosts.https=false \
	--set global.ingress.configureCertmanager=false \
	--set global.ingress.class=nginx \
	--set certmanager.install=false \
	--set global.externalUrl=http://localhost:8080 \
	--set gitlab.webservice.service.externalPort=8080 \
	--set gitlab.webservice.assets.enabled=true \
	--set gitlab-runner.install=false \
	--timeout 600s
	

 
install_argocd:
	kubectl apply -R -f ./config/
	kubectl get namespaces
	kubectl apply -f gitlab-creds-secret.yaml     
	kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
	kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
	kubectl -n argocd rollout status deployment argocd-server
	kubectl get pods -n argocd
	kubectl apply -f ./app/my-app-argocd.yaml


uninstall_gitlab:
	helm uninstall gitlab -n gitlab || true
	kubectl delete namespace gitlab || true

# Username for gitlab is root and for argocd is admin
password:
	kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
	kubectl get secret gitlab-gitlab-initial-root-password -n gitlab -ojsonpath="{.data.password}" | base64 --decode

expose_argocd:
	kubectl port-forward svc/argocd-server -n argocd 8081:80

# use 8181 instead of 8080 or css wont load
expose_gitlab:
	kubectl port-forward svc/gitlab-webservice-default -n gitlab 8080:8181

verify:
	kubectl get pods -n gitlab
	curl loc:8888

	
# show_app_in_argocd:
# 	argocd login localhost:8080
# 	argocd proj list
# 	argocd app list


#create_app:
#	argocd login localhost:8080
#	argocd app create my-app
#	  --repo git@github.com:0xOnyx/argocd-demo.git \
#      --path ./ \
#      --dest-server https://kubernetes.default.svc \
#      --dest-namespace dev