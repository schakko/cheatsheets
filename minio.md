# Minio
## Ingress for Kubernetes

	apiVersion: extensions/v1beta1
	kind: Ingress
	metadata:
	  name: minio-ingress
	  annotations:
		nginx.ingress.kubernetes.io/proxy-body-size: 1024m
	spec:
	  rules:
	  - host: minio.domain.tld
		http:
		  paths:
		  - path: /
			backend:
			  serviceName: minio
			  servicePort: http
			  

## Create a bucket and restrict access for user to this bucket

	# create bucket
	docker pull minio/mc
	docker run -it --entrypoint=/bin/sh minio/mc
	
	mc mb ${BUCKET_NAME}
	mc config host add minio https://minio.domain.tld ${ACCESS_KEY} ${SECRET_KEY}
	
	cat > restrict-bucket-access.json << EOF
	{
	  "Version": "2012-10-17",
	  "Statement": [
		{
		  "Action": [
			"s3:GetObject"
		  ],
		  "Effect": "Allow",
		  "Resource": [
			"arn:aws:s3:::${BUCKET_NAME}/*"
		  ],
		  "Sid": ""
		}
	  ]
	}
	EOF
	
	mc admin policy add minio jenkins-spinnaker-deployment-access restrict-bucket-access.json
	mc admin user add minio jenkins-spinnaker-user ${SECRET_KEY} jenkins-spinnaker-deployment-access
	# ACCESS_KEY=jenkins-spinnaker-user
