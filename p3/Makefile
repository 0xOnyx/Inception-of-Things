.PHONY: all clean create_cluseter install

all: clean create_cluster install

create_cluster:
	k3d cluster create p3 -p 8443:443@loadbalancer -p 8888:8888@loadbalancer
	k3d kubeconfig get p3

install:
	kubectl apply -R -f ./config/
	kubectl get namespaces
	kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
	kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
	kubectl -n argocd rollout status deployment argocd-server
	kubectl get pods -n argocd
	kubectl apply -f ./app/my-app-argocd.yaml

password:
	kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}"


clean:
	k3d cluster delete p3

show_app_in_argocd:
	argocd login localhost:8080
	argocd proj list
	argocd app list


verify:
	kubectl get pods -n dev
	curl http://localhost:8888


expose_argocd:
	kubectl port-forward svc/argocd-server -n argocd 8081:80

#create_app:
#	argocd login localhost:8080
#	argocd app create my-app
#	  --repo git@github.com:0xOnyx/argocd-demo.git \
#      --path ./ \
#      --dest-server https://kubernetes.default.svc \
#      --dest-namespace dev