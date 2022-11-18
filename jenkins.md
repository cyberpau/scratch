
## Retrieve jenkins password:

```
printf $(kubectl get secret cd-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo
```

## Configure Jenkins Cloud for Kubernetes

1. In the Jenkins user interface, select Manage Jenkins > Manage nodes and clouds.
2. Click Configure Clouds in the left navigation pane.
3. Click Add a new cloud and select Kubernetes.
4. Click Kubernetes Cloud Details.
5. In the Jenkins URL field, enter the following value: `http://cd-jenkins:8080`
6. In the Jenkins tunnel field, enter the following value: `cd-jenkins-agent:50000`
7. Click Save.

## Get credentials
```
gcloud container clusters get-credentials jenkins-cd --zone us-east1-b --project qwiklabs-gcp-00-95663d5b9aa1
```

## Jenkins port-forward

```
export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/component=jenkins-master" -l "app.kubernetes.io/instance=cd" -o jsonpath="{.items[0].metadata.name}")
kubectl port-forward $POD_NAME 8080:8080 >> /dev/null &
```
