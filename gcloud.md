# GCloud

## Config
Set Config: [https://cloud.google.com/sdk/gcloud/reference/config/set](https://cloud.google.com/sdk/gcloud/reference/config/set)

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




