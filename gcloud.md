# GCloud

## Setup
List of accounts
```
gcloud auth list
```

List of projects
```
gcloud config list project
```

## Config
Set Config: [https://cloud.google.com/sdk/gcloud/reference/config/set](https://cloud.google.com/sdk/gcloud/reference/config/set)

Examples:
```
gcloud config set compute/region us-west1

```

## Google Cloud Storage
Make bucket (`mb`)
```
gsutil mb <YOUR-BUCKET-NAME> // gsutil mb gs://qwiklabs-gcp-00-846807476a94
```

Copy files to bucket
```
gsutil cp ada.jpg gs://YOUR-BUCKET-NAME
```

Copy files from bucket
```
gsutil cp -r gs://YOUR-BUCKET-NAME/ada.jpg .
```

List files
```
gsutil ls gs://YOUR-BUCKET-NAME
```

Change Access Control List
```
gsutil acl ch -u AllUsers:R gs://YOUR-BUCKET-NAME/ada.jpg
```

Delete
```
gsutil rm gs://YOUR-BUCKET-NAME/ada.jpg
```

## Functions
Deploy
```
gcloud functions deploy helloWorld \
  --stage-bucket [BUCKET_NAME] \
  --trigger-topic hello_world \
  --runtime nodejs8
```

Check the status
```
gcloud functions describe helloWorld
```

View logs
```
gcloud functions logs read helloWorld
```



## Monitoring
Installing agent
```
curl -sSO https://dl.google.com/cloudagents/add-google-cloud-ops-agent-repo.sh

sudo bash add-google-cloud-ops-agent-repo.sh --also-install

```

## PubSub

Create a new pubsub topic
```
gcloud pubsub topics create new-lab-report
```


Publish a message:
```
async function publishPubSubMessage(labReport) {
  const buffer = Buffer.from(JSON.stringify(labReport));
  await pubsub.topic('new-lab-report').publish(buffer);
}

```

Get / Pull message after publish:
```
gcloud pubsub subscriptions pull --auto-ack MySub
```

## Service Account

Create pubsub service account
```
gcloud iam service-accounts create pubsub-cloud-run-invoker --display-name "PubSub Cloud Run Invoker"

```

Give the service account permission to invoke email service
```
gcloud run services add-iam-policy-binding email-service --member=serviceAccount:pubsub-cloud-run-invoker@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com --role=roles/run.invoker --region us-east1 --platform managed

```
## Cloud Run

Get project number
```
PROJECT_NUMBER=$(gcloud projects list --filter="qwiklabs-gcp" --format='value(PROJECT_NUMBER)')
```

Enable the project to create pubsub auth token
```
gcloud projects add-iam-policy-binding $GOOGLE_CLOUD_PROJECT --member=serviceAccount:service-$PROJECT_NUMBER@gcp-sa-pubsub.iam.gserviceaccount.com --role=roles/iam.serviceAccountTokenCreator
```


Create pubsub subscription
```
gcloud pubsub subscriptions create email-service-sub --topic new-lab-report --push-endpoint=$EMAIL_SERVICE_URL --push-auth-service-account=pubsub-cloud-run-invoker@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
```

Create Dockerfile and build:
```
docker build -t <service-name>:<version> .
```

Build via Cloud Build and deploy container via Cloud Run (deploy.sh)
```
gcloud builds submit \
  --tag gcr.io/$GOOGLE_CLOUD_PROJECT/email-service
gcloud run deploy email-service \
  --image gcr.io/$GOOGLE_CLOUD_PROJECT/email-service \
  --platform managed \
  --region us-east1 \
  --no-allow-unauthenticated \
  --max-instances=1
  --service-account billing-service-sa-408@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
```

Get URL of a service
```
BILLING_URL=$(gcloud run services describe private-billing-service-622 \
--platform managed \
--region us-central1 \
--format "value(status.url)")
```

## Google Kubernetes Engine

Create GKE Cluster:
```
gcloud container clusters create --machine-type=e2-medium --zone=us-east1-b lab-cluster 

```

Authenticate with cluster
```
gcloud container clusters get-credentials lab-cluster 
```

Deploy an application to the cluster
```
kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0
```

Expose to external traffice:
```
kubectl expose deployment hello-server --type=LoadBalancer --port 8080
```

Create a firewall rule to allow external traffic to the VM instances:
```
gcloud compute firewall-rules create www-firewall-network-lb \
    --target-tags network-lb-tag --allow tcp:80
```


Create static external ip for the load balancer:
```
   gcloud compute addresses create network-lb-ip-1 \
    --region us-central1 
    
```

Add legacy http health check resource:
```
gcloud compute http-health-checks create basic-check

```

Add a target pool in the same region as your instances. Run the following to create the target pool and use the health check, which is required for the service to function:
```
  gcloud compute target-pools create www-pool \
    --region us-east4 --http-health-check basic-check
```


Add instances to the pool:
```
gcloud compute target-pools add-instances www-pool \
    --instances www1,www2,www3
```

Add forwarding rule:
```
gcloud compute forwarding-rules create www-rule \
    --region  us-east4 \
    --ports 80 \
    --address network-lb-ip-1 \
    --target-pool www-pool
```

Enter the following command to view the external IP address of the www-rule forwarding rule used by the load balancer:
```
gcloud compute forwarding-rules describe www-rule --region us-east4
```


Create a loadbalancer template:
```
gcloud compute instance-templates create lb-backend-template \
   --region=us-east4 \
   --network=default \
   --subnet=default \
   --tags=allow-health-check \
   --machine-type=e2-medium \
   --image-family=debian-11 \
   --image-project=debian-cloud \
   --metadata=startup-script='#!/bin/bash
     apt-get update
     apt-get install apache2 -y
     a2ensite default-ssl
     a2enmod ssl
     vm_hostname="$(curl -H "Metadata-Flavor:Google" \
     http://169.254.169.254/computeMetadata/v1/instance/name)"
     echo "Page served from: $vm_hostname" | \
     tee /var/www/html/index.html
     systemctl restart apache2'
```

Add additional users to project:
```
  gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member=user:[EMAIL] --role=roles/logging.viewer
  
  gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member=user:[EMAIL] --role roles/source.writer
  
```

Setting cloud shell environment variables, to apply, use `source ./infraclass/config`. To create persistence, save it to `.profile` by `vi .profile`
```
echo INFRACLASS_PROJECT_ID=<INSERT_PROJECT_ID> >> ~/infraclass/config
echo INFRACLASS_REGION=<INSERT_REGION> >> ~/infraclass/config
```
