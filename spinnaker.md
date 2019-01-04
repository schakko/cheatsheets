# Installation
- ~/.kube must contain valid Kubernetes credentials

## Deploy Minio

See https://www.minio.io/kubernetes.html

## Install Halyard in Docker
Be aware of wrong file permission with Docker in WSL!

	docker run -p 8084:8084 -p 9000:9000 \
		--name halyard --rm \
		-v ~/.hal:/home/spinnaker/.hal \
		-v ~/.kube:/home/spinnaker/.kube
		-it \
		gcr.io/spinnaker-marketplace/halyard:stable
		
	# another shell
	docker exec -it halyard bash
	source <(hal --print-bash-completion)
	
## Kubernetes Provider V2 (Manifest Based) 

	hal config provider kubernetes enable
	
	CONTEXT=$(kubectl config current-context)
	
	hal config provider kubernetes account add ${YOUR_KUBERNETES_ACCOUNT} \
    --provider-version v2 \
    --context $CONTEXT
	
	hal config features edit --artifacts true
	
## Storage Service: Minio
Fetch Minio endpoint URL from Kubernetes logs and MINIO_ACCESS_KEY and MINIO_SECRET_KEY from Kubernetes Minio environment


	export ENDPOINT=
	export MINIO_ACCESS_KEY=
	export MINIO_SECRET_KEY=
	
	mkdir -p ~/.hal/default/profiles
	echo "spinnaker.s3.versioning: false" > ~/.hal/default/profiles/front50-local.yml
	
	echo $MINIO_SECRET_KEY | hal config storage s3 edit --endpoint $ENDPOINT \
	--access-key-id $MINIO_ACCESS_KEY \
	--secret-access-key # will be read on STDIN to avoid polluting your 
	# ~/.bash_history with a secret

	hal config storage edit --type s3


## Expose on public port
See https://blog.spinnaker.io/exposing-spinnaker-to-end-users-4808bc936698

	echo "host: 0.0.0.0" | tee \
	~/.hal/default/service-settings/gate.yml \
	~/.hal/default/service-settings/deck.yml
	
	hal deploy apply

## Add nginx ingress

	apiVersion: extensions/v1beta1
	kind: Ingress
	metadata:
	  name: spin-ingress
	  namespace: spinnaker
	  annotations:
	    ingress.kubernetes.io/rewrite-target: /
	spec:
	  rules:
	  - host: spinnaker.sub.domain.com
	    http:
	      paths:
	      - path: /*
		backend:
		  serviceName: spin-deck
		  servicePort: 9000
	      - path: /gate/*
		backend:
		  serviceName: spin-gate
		  servicePort: 8084
	---
	# Addresses issue https://github.com/spinnaker/spinnaker/issues/1934
	apiVersion: extensions/v1beta1
	kind: Ingress
	metadata:
	  name: spin-auth-ingress
	  namespace: spinnaker
	  annotations:
	    ingress.kubernetes.io/rewrite-target: /auth/
	spec:
	  rules:
	  - host: spinnaker.sub.domain.com
	    http:
	      paths:
	      - path: /auth/*
		backend:
		  serviceName: spin-gate
		  servicePort: 8084

## Deploy

	hal version list
	hal config version edit --version ${VERSION}
	hal deploy apply
	
	# see k8 status for deployment
	
## Backup

	hal backup create
	cp ~/.hal/*.tar /to/any/writeable/directory/in/docker/container
	
