.PHONY: all clean create_cluseter install

all: clean create_cluster install

create_cluster:
	k3d cluster create p3 --port 8888:8888@loadbalancer
	k3d kubeconfig get p3

install:
	kubectl apply -R -f ./config/
	kubectl get namespaces
	kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
#kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
	kubectl get pods -n argocd
	kubectl apply -f ./app/my-app-argocd.yaml

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