# managed kubernetes @ionos-enterprise

About kubernetes@ionos: https://www.ionos.de/enterprise-cloud/managed-kubernetes

This is just a kickstart guideline how to setup a basic k8s-cluster with an ingress controller and cert-manager.

## initial setup

### 1st: cluster setup at DCD
* login with your account at https://dcd.ionos.com/
* create a managed kubernetes
* add a node pool to the cluster
* create a static ip
  * if you don't create this ip and provide it for ingress, k8s will create it and will get lost if the cluster will change / deleted / recreated
* download the `kubeconfig.yaml`
* set the downlaoded config as your default (`export KUBECONFIG=/path/to/downloaded/kubeconfig.yaml`)

### 2nd: install basic tools

Manage request to the services.

### ingress controller
* install ingress controller via helm:
  ```sh
  helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
  helm install ingress-nginx ingress-nginx/ingress-nginx --set controller.service.loadBalancerIP=1.2.3.4
  ```
  * replace `1.2.3.4` with the static ip from step 1
  * configuration options during the setup https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-helm/

### cert-manager

Let's encrypt support for your http traffic.

* install cert-manager via helm
  ```sh
  kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v0.15.1/cert-manager.crds.yaml
  kubectl create namespace cert-manager
  helm install cert-manager --namespace cert-manager jetstack/cert-manager
  ```
* create a cluster issuer for the `cert-manager`:
  ```yaml
  apiVersion: cert-manager.io/v1alpha2
  kind: ClusterIssuer
  metadata:
    name: letsencrypt-prod
  spec:
    acme:
      # You must replace this email address with your own.
      # Let's Encrypt will use this to contact you about expiring
      # certificates, and issues related to your account.
      email: ssl@example.com
      server: https://acme-v02.api.letsencrypt.org/directory
      privateKeySecretRef:
        # Secret resource used to store the account's private key.
        name: letsencrypt-prod-key
      # Add a single challenge solver, HTTP01 using nginx
      solvers:
      - http01:
          ingress:
            class: nginx
  ```
  * some more information about the issuer: https://cert-manager.io/docs/concepts/issuer/

## demo cluster

Justs clone this repository and replace `example.org` with your domain and apply in this order:

```sh
# clone and enter the project folder
git clone git@github.com:Loumaris/kubernetes-ionos-example.git
cd kubernetes-ionos-example

# create a simple echo service
kubectl apply -f demo/001-echo-pod.yaml

# create a simple whoami-service
kubectl apply -f demo/002-whoami-deployment.yaml

# setup ingress to get the domain up and running (without tls)
kubectl apply -f demo/003-ingress-without-tls.yaml

# create a let's encrypt certificate
kubectl apply -f demo/004-certificate.yaml

# setup ingress with the new certificate
kubectl apply -f demo/005-ingress-with-tls.yaml
```

Now you can reach `https://example.org` and `https://example.org/who` :) Feel free to add new services ;-)

## update a service inside the cluster

### option 1: change the yaml

You can alway update the yaml and apply it via
`kubectl apply -f <service.deployment.yml>`

### option 2: patch via kubectl

For CI/CD you can change the image via kubectl directly:
`kubectl set image deployment/<service_name> <service_name>=$CONTAINER_RELEASE_IMAGE --record`


# Contributing

Contributions are most welcome!

This example is just getting started, please contribute to make it super awesome.