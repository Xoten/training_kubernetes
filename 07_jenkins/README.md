

Create a namespace and a volume
Before you install the Jenkins chart, you need to create two things:

A new namespace to logically segregate your Jenkins objects within the cluster
A persistent volume to store Jenkins configuration, even if you restart Minikube

kubectl apply -f jenkins-ns.yaml

And do the same again for the persistent volume, as follows.

kubectl apply -f jenkins-pv.yaml

Installing Jenkins
You’ll install Jenkins with the community Helm chart. This chart is pretty complex; if you’re interested, you can explore it on Github: https://github.com/helm/charts/tree/master/stable/jenkins.

First, create a values.yml file. Helm will interpolate the following code into the Jenkins chart to set appropriate defaults for running on Minikube.

Now, to install Jenkins, run the following helm command:

```
helm repo add jenkins https://charts.jenkins.io
helm repo update
helm install
  --name jenkins 
  --namespace jenkins
  --values values.yml 
  jenkins/jenkins 
```

1. Get your 'admin' user password by running:
  kubectl exec --namespace jenkins -it svc/jenkins -c jenkins -- /bin/cat /run/secrets/additional/chart-admin-password && echo
2. Get the Jenkins URL to visit by running these commands in the same shell:
  export NODE_PORT=$(kubectl get --namespace jenkins -o jsonpath="{.spec.ports[0].nodePort}" services jenkins)
  export NODE_IP=$(kubectl get nodes --namespace jenkins -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT/login

3. Login with the password from step 1 and the username: admin

## RBAC set up
Log in to the Jenkins dashboard.
Navigate to Credentials > System > Global Credentials > Add Credentials.
Add a Kubernetes Service Account credential, setting the value of the ID field to jenkins.
Save and navigate to Jenkins > Manage Jenkins > System.
Under the Kubernetes section, configure the credentials to those you created in step 3 and click Save.

## Testing it all works

You can run a simple build to make sure everything’s working. First, log in to your new Jenkins dashboard and navigate to New Item in the left-hand column.

