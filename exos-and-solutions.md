# Container Orchestration with Kubernetes — Exercises & Solutions

**Repository to use:**  
[GitLab – kubernetes-exercises](https://gitlab.com/twn-devops-bootcamp/latest/10-kubernetes/kubernetes-exercises)

Your company's java-mysql application is running with docker-compose on a server. This application is used often internally and by your company clients too. You noticed that the server isn't very stable: Often a database container dies or the application itself, or docker daemon must be restarted. During this time people can't access the app!

So when this happens, the users write to you to tell you that the app is down and ask you to fix it. You SSH into the server, restart the containers with docker-compose and containers start again.

But this is annoying work, plus it doesn't look good for your company that your clients often can't access the app. So you want to make your application more reliable and highly available. You want to replicate both the database and the app, so if one container goes down, there is always a backup. Also you don't want to rely on a single server, but have multiple, in case 1 whole server goes down or gets rebooted etc.

So you look into different solutions and decide to use the container orchestration tool Kubernetes to solve the issue. For now you want to configure it and deploy your application manually, since it's a new tool and want to try it out manually before automating.

***

## Exercise 1: Create a Kubernetes cluster
**Task:**  
Create a Kubernetes cluster (Minikube or LKE)

**Solution:**  
**Minikube:**
```sh
minikube start
```

**LKE:**
```sh
# On Linode UI dashboard, create K8s cluster with 2 smallest nodes "Dedicated 4 GB" plan
# Update your KUBECONFIG context to connect to Linode environment
```

***

## Exercise 2: Deploy Mysql with 2 replicas
**Task:**  
First of all, you want to deploy the mysql database.
* Deploy Mysql database with 2 replicas and volumes for data persistence
* To simplify the process you can use Helm for this.

**Solution:**  
*General notes:*
- All the k8s manifest files for the exercise are in "k8s-deployment" folder, so:
```sh
# clone this repository locally
git clone https://gitlab.com/twn-devops-bootcamp/latest/10-kubernetes/kubernetes-exercises.git

# check out the solutions branch
git checkout feature/solutions

# change to k8s-deployment folder
cd k8s-deployment
```

- Mysql Chart link: https://github.com/bitnami/charts/tree/master/bitnami/mysql 

**Minikube:**
```sh
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-release bitnami/mysql -f mysql-chart-values-minikube.yaml
```

**LKE:**
```sh
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-release bitnami/mysql -f mysql-chart-values-lke.yaml
```

***

## Exercise 3: Deploy your Java Application with 2 replicas
**Task:**  
Now you want to:
* Deploy the Java application with 2 replicas - note: once you have pushed your image to docker hub, you will need to ensure you are using this image name and tag in your application's configuration
* With docker-compose, you were setting env_vars on the server. In K8s there are separate components for that, so you want to:
* Create ConfigMap and Secret with the correct values and reference them in the application deployment config file.

**Solution:**  
**Minikube & LKE:**
```sh
# Create my-registry-key secret to pull image
DOCKER_REGISTRY_SERVER=docker.io
DOCKER_USER=your dockerID, same as for `docker login`
DOCKER_EMAIL=your dockerhub email, same as for `docker login`
DOCKER_PASSWORD=your dockerhub pwd, same as for `docker login`

kubectl create secret docker-registry my-registry-key \
--docker-server=$DOCKER_REGISTRY_SERVER \
--docker-username=$DOCKER_USER \
--docker-password=$DOCKER_PASSWORD \
--docker-email=$DOCKER_EMAIL

# Again from k8s-deployment folder, execute following commands - ensuring you have added your docker image details to line 22 of java-app.yaml
kubectl apply -f db-secret.yaml
kubectl apply -f db-config.yaml
kubectl apply -f java-app.yaml
```

***

## Exercise 4: Deploy phpmyadmin
**Task:**  
As a next step you:
* Deploy phpmyadmin to access Mysql UI.
* For this deployment you just need 1 replica, since this is only for your own use, so it doesn't have to be Highly Available. A simple deployment.yaml file and internal service will be enough.

**Solution:**  
**Minikube & LKE:**
```sh
kubectl apply -f phpmyadmin.yaml
```

***

Now your application setup is running in the cluster, but you still need a proper way to access the application. Also, you don't want users to access the application using the IP address but instead to use a domain name. For that, you want to install Ingress controller in the cluster and configure ingress access for your application.

***

## Exercise 5: Deploy Ingress Controller
**Task:**  
Deploy Ingress Controller in the cluster - using Helm

**Solution:**  
**Minikube:**
```sh
# minikube comes with ingress addon, so we just need to activate it
minikube addons enable ingress 
```

**LKE:**
```sh
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install ingress-nginx ingress-nginx/ingress-nginx
```

*Notes on installing Ingress-controller on LKE:*
- Chart link: https://github.com/kubernetes/ingress-nginx/tree/main/charts/ingress-nginx

***

## Exercise 6: Create Ingress rule
**Task:**  
Create an Ingress rule for your application's access.
* If you are using Minikube, the application must be accessible on my-java-app.com
* For LKE, use the Linode node-balancer address - line 48 of the Index.html file for the application must be updated to include the address

**Solution:**  
**Minikube:**
- Set the host name in java-app-ingress.yaml line 7 to my-java-app.com
- Add `127.0.0.1 my-java-app.com` in /etc/hosts file
- Create ingress component: `kubectl apply -f java-app-ingress.yaml`
- Run `minikube tunnel` command in terminal window
- Access application from browser on address: `my-java-app.com`

**LKE:**
- Set the HOST variable found at line 48 of the index.html to the Linode node-balancer address (you may need to rebuild your container image after this step)
- Set the host name in java-app-ingress.yaml line 7 to Linode node-balancer address
- Create ingress component: `kubectl apply -f java-app-ingress.yaml`
- Access application from browser on Linode node-balancer address

***

## Exercise 7: Port-forward for phpmyadmin
**Task:**  
However, you don't want to expose phpmyadmin for security reasons. So you configure port-forwarding for the service to access on localhost, whenever you need it.
* Configure port-forwarding for phpmyadmin

**Solution:**  
**Minikube & LKE:**
```sh
kubectl port-forward svc/phpmyadmin-service 8081:8081
```

***

As the final step, you decide to create a helm chart for your Java application where all the configuration files are configurable. You can then tell developers how they can use it by setting all the chart values. This chart will be hosted in its own git repository.

***

## Exercise 8: Create Helm Chart for Java App
**Task:**  
* All config files: service, deployment, ingress, configMap, secret, will be part of the chart
* Create custom values file as an example for developers to use when deploying the application
* Deploy the java application using the chart with helmfile
* Host the chart in its own git repository

**Solution:**  
**Steps:**

- Create helm chart boilerplate for your application with chart-name `java-app` using command: `helm create java-app`

***Note**: This will generate `java-app` folder with chart files*

- Clean up all unneeded contents from `java-app` folder, as you learned in the module
- Create template files for `db-config.yaml`, `db-secret.yaml`, `java-app-deployment.yaml`, `java-app-ingress.yaml`, `java-app-service.yaml`
- Create `values-override.yaml` and set all the correct values there 
- Set default chart values in `values.yaml` file

:exclamation: **Check the final version of chart files in `java-app` folder in this `feature/solutions` branch**

***Note**: the `ingress.hostName` must be set to `my-java-app.com` for Minikube & Linode node balancer address*

- Validate that your chart is correct and debug any issues, do a dry-run:

```sh
helm install my-cool-java-app java-app -f java-app/values-override.yaml --dry-run --debug
```

- If dry-run shows the k8s manifest files with correct values, everything is working, so you can create the chart release:

```sh
helm install my-cool-java-app java-app -f java-app/values-override.yaml 
```

- Extract the chart `java-app` folder and host into its own new git repository `java-app-chart`
