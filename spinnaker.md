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
	
	hal config security ui edit \
		--override-base-url https://spinnaker.sub.domain.com

	hal config security api edit \
		--override-base-url https://spinnaker-api.sub.domain.com
		
	hal deploy apply


## Add nginx ingress
See https://medium.com/parkbee-tech/spinnaker-installation-on-kubernetes-using-new-halyard-based-helm-chart-d0cc7f0b8fd0

	# paste this to spinnaker-nginx-ingress.yml
	
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
	      - path: /
		backend:
		  serviceName: spin-deck
		  servicePort: 9000
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
	  - host: spinnaker-api.sub.domain.com
	    http:
	      paths:
	      - path: /
		backend:
		  serviceName: spin-gate
		  servicePort: 8084


Apply this deployment:

	kubectl apply -f spinnaker-nginx-ingress.yml --namespace spinnaker

## Configure providers
### Docker registry (e.g. Nexus)

	hal config provider docker-registry account add docker --address https://registry.sub.domain.com --username=${USERNAME} --password
	# password must be entered
	
## Deploy

	hal version list
	hal config version edit --version ${VERSION}
	hal deploy apply
	
	# see k8 status for deployment
	
## Backup

	hal backup create
	cp ~/.hal/*.tar /to/any/writeable/directory/in/docker/container
	
# Halyard
## Latest Docker image for Halyard

The latest Halyard Docker image is tagged with "nightly". A full list of images is available at https://console.cloud.google.com/gcr/images/spinnaker-marketplace/GLOBAL/halyard

# Bugs
## Wrong .kube/config file after restoring a halyard backup

After running `hal backup restore --backup-path=${MY_BACKUP}` the `hal deploy apply` command failed with

	! ERROR Failed check for Namespace/spinnaker in null
	Error in configuration: context was not found for specified context: ${VALID_CONTEXT_NAME}
	
I could verify with help of

	kubectl config get-contexts
	kubectl get ns
	
that the context was valid and could be used.

When checking halyard daemon's log, I realized that

	2019-01-08 22:40:24.836  INFO 6 --- [tionScheduler-1] c.n.s.h.core.job.v1.JobExecutorLocal     : Executing 95d3f6f9-626f-4ed3-8cb5-03b642b853f4with tokenized command: [kubectl, --context, ${VALID_CONTEXT_NAME}, --kubeconfig, /home/spinnaker/.hal/default/staging/dependencies/603017638-2054179555-config, get, Namespace, spinnaker]
	2019-01-08 22:40:25.535  INFO 6 --- [      Thread-12] c.n.s.h.core.job.v1.JobExecutorLocal     : 95d3f6f9-626f-4ed3-8cb5-03b642b853f4 has terminated with exit code 1
	2019-01-08 22:40:25.538  INFO 6 --- [      Thread-12] c.n.s.h.core.tasks.v1.TaskRepository     : Task [Apply deployment] (a6d99382-7029-43f2-a45d-492e33a9206c) - RUNNING failed with HalException:

	com.netflix.spinnaker.halyard.core.error.v1.HalException: Failed check for Namespace/spinnaker in null
	Error in configuration: context was not found for specified context: ${VALID_CONTEXT_NAME}


		at com.netflix.spinnaker.halyard.deploy.spinnaker.v1.service.distributed.kubernetes.v2.KubernetesV2Utils.exists(KubernetesV2Utils.java:101) ~[halyard-deploy.jar:na]
		at com.netflix.spinnaker.halyard.deploy.spinnaker.v1.service.distributed.kubernetes.v2.KubernetesV2Utils.exists(KubernetesV2Utils.java:62) ~[halyard-deploy.jar:na]
		at com.netflix.spinnaker.halyard.deploy.deployment.v1.KubectlDeployer.lambda$deploy$0(KubectlDeployer.java:69) ~[halyard-deploy.jar:na]
		at java.util.ArrayList$ArrayListSpliterator.forEachRemaining(ArrayList.java:1382) ~[na:1.8.0_181]
		at java.util.stream.ReferencePipeline$Head.forEach(ReferencePipeline.java:580) ~[na:1.8.0_181]
		at com.netflix.spinnaker.halyard.deploy.deployment.v1.KubectlDeployer.deploy(KubectlDeployer.java:45) ~[halyard-deploy.jar:na]
		at com.netflix.spinnaker.halyard.deploy.deployment.v1.KubectlDeployer.deploy(KubectlDeployer.java:37) ~[halyard-deploy.jar:na]
		at com.netflix.spinnaker.halyard.deploy.services.v1.DeployService.deploy(DeployService.java:287) ~[halyard-deploy.jar:na]
		at com.netflix.spinnaker.halyard.controllers.v1.DeploymentController.lambda$deploy$14(DeploymentController.java:210) ~[halyard-web.jar:na]
		at com.netflix.spinnaker.halyard.core.DaemonResponse$StaticRequestBuilder.build(DaemonResponse.java:127) ~[halyard-core.jar:na]
		at com.netflix.spinnaker.halyard.core.tasks.v1.TaskRepository.lambda$submitTask$1(TaskRepository.java:48) ~[halyard-core.jar:na]
		at java.lang.Thread.run(Thread.java:748) ~[na:1.8.0_181]
		
used the previously compiled configuration (`603017638-2054179555-config`), which in turn was compiled by halyard using the `~/.hal/config` file. Because of restoring the backup, the `kubeconfigFile` parameter in `~/.hal/config` has been resetted and pointed to a non-existing kube configuration, resulting in a missing context:

    kubernetes:
        kubeconfigFile: /home/spinnaker/.hal/.backup/required-files/2054179555-config

I fixed the issue by simply resetting the kubeconfig-file to the default location and re-applying the deployment

	hal config provider kubernetes account edit my_account --kubeconfig-file=/home/spinnaker/.kube/config
	hal deploy apply
