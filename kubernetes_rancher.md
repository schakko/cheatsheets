# Docker
## Execute Bash inside container

	docker exec -it kubelet /bin/bash

## List mount points
  
	# list container id
	docker ps
	# show mount points
	docker inspect -f '{{ .Mounts }}' ${CONTAINER_ID}

# RancherOS
## Instant Upgrade RancherOS

	# show current version
	sudo ros os version
	# view available versions
	sudo ros list
	# upgrade
	sudo ros upgrade
  
# Kubernetes
## Show logs for kubelet or kube-proxy

	docker logs -f kubelet
	docker logs -f kube-proxy
## Show secrets

	kubectl get secrets --namespace ${NAMESPACE}
	kubectl get secrets/${SECRET_NAME} --namespace ${NAMESPACE} -o yaml
	base64 --decode ${SECRET_KEY}
	# one-liner:
	# kubectl get secrets/${SECRET_NAME} --namespace ${NAMESPACE} -o yaml | grep ${SECRET_KEY} | sed 's/\s\s*/ /g' | cut -d' ' -f3 | base64 --decode

## Create Namespace

Create file namespace-demo.yml

	{
	  "kind": "Namespace",
	  "apiVersion": "v1",
	  "metadata": {
	    "name": "demo",
	    "labels": {
	      "name": "demo"
	    }
	  }
	}

Execute

	kubectl create -f namespace-demo.yml
	
# Rancher
## Download API keys
- Log in to Rancher
- Go to *Cluster*
- Click *Kubeconfig.file*
- Paste content to *~/.kube/kubeconfig*
