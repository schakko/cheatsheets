# Docker
## Execute Bash inside container

	docker exec -it kubelet /bin/bash

## List volumes
  
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
  
# Rancher
## Download API keys
- Log in to Rancher
- Go to *Cluster*
- Click *Kubeconfig.file*
- Paste content to *~/.kube/kubeconfig*
