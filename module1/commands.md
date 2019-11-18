# These instructions assume that a cluster is already available to you
# Ensure that you can connect to the cluster and that kubectl 1.15+ is installed

# Windows https://storage.googleapis.com/kubernetes-release/release/v1.15.0/bin/windows/amd64/kubectl.exe
curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.15.0/bin/windows/amd64/kubectl.exe

# Linux
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl

# Mac
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl

kubectl version --short

# Install booksapp
## Create and use the namespace
kubectl create ns booksapp

kubectl config set-context $(kubectl config current-context) --namespace=booksapp

# Clone the repository
git clone https://github.com/cpretzer/linkerd-traffic-split
cd linkerd-traffic-split

# Installing booksapp
kubectl apply -f module1/booksapp.yml

# watch the pods start up
kubectl get po -w

# verify that the service has an external endpoint this may take a few minutes
kubectl get svc -w

# get the external URL and verify that the page loads
kubectl get svc webapp -n booksapp -ojsonpath={.status.loadBalancer.ingress[].ip}

# use the output from the command above in the URL of your browser
http://<output from command above>:7000

## install linkerd cli

## Linux and Mac
curl -sL https://run.linkerd.io/install | sh

## Windows
# Download https://github.com/linkerd/linkerd2/releases/download/stable-2.6.0/linkerd2-cli-stable-2.6.0-windows.exe
# and add it to your PATH environment variable as "linkerd.exe"

# verify the cli installation
linkerd version

# run a pre-check
linkerd check --pre

# view the linkerd control plane resources
linkerd install

# deploy linkerd control plane resources to the cluster
linkerd install | kubectl apply -f -

# check the linkerd control plane resources
linkerd check

# view the control plane resources in the linkerd namespace
kubectl get po -n linkerd
kubectl get deploy -n linkerd
kubectl get svc -n linkerd

# open the dashboard
# Note: this will run the command in the background, and will need to be killed later
linkerd dashboard &

# update the booksapp namespace to enable automatic proxy injection
kubectl annotate ns booksapp linkerd.io/inject=enabled

# restart the deployments to inject the pods
kubectl rollout restart deploy -n booksapp 

# verify that the pods start and that there are two containers per pod
kubectl get po -w -n booksapp

# use linkerd tap
linkerd tap -n booksapp deploy/webapp -o wide
linkerd routes -n booksapp deploy/webapp --to svc/books -o wide
linkerd edges -n booksapp deploy

# create service profiles

# webapp
## On the browser, click "Namespaces" on the left navbar
## Click booksapp under "HTTP metrics"
## Click webapp
## Click the Route Metrics tab
linkerd profile --open-api module1/webapp.swagger -n booksapp webapp > webapp-sp.yml
kubectl apply -f module1/webapp-sp.yml

# Observe that there are new routes in the table

# books
## On the browser, click "Namespaces" on the left navbar
## Click booksapp under "HTTP metrics"
## Click books
## Click the Route Metrics tab
linkerd profile --open-api module1/books.swagger -n booksapp books > module1/books-sp.yml
kubectl apply -f module1/books-sp.yml

# Observe that there are new routes in the table

# authors
## On the browser, click "Namespaces" on the left navbar
## Click booksapp under "HTTP metrics"
## Click authors
## Click the Route Metrics tab
linkerd profile --open-api module1/authors.swagger -n booksapp authors > module1/authors-sp.yml
kubectl apply -f module1/authors-sp.yml

# Observe that there are new routes in the table

# Install the ingress

## ingress controller deployment
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/mandatory.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/cloud-generic.yaml

# ensure that the ingress controller pod is running
kubectl get po -n ingress-nginx

# deploy the ingress service
kubectl apply -f module1/nginx-ingress-svc.yml 

# view the service
kubectl get svc ingress-nginx -n ingress-nginx -o wide

# deploy the ingress resource
kubectl apply -f module1/books-ingress.yml

# get the ingress External IP address and make sure that it works
kubectl get svc -w -n ingress-nginx # until the EXTERNAL-IP address is no longer <pending>
kubectl get svc ingress-nginx -n ingress-nginx -ojsonpath={.status.loadBalancer.ingress[].ip}

# use the output from the previous command in the address bar of your browser
## Note: we are currently using http and not https
http://<output from previous command>

# Go to the linkerd dashboard tab in your browser

# View the injected ingress controller deployment yaml
## This does not change the nginx ingress deployment
kubectl get deploy -n ingress-nginx nginx-ingress-controller -oyaml | linkerd inject -

# Apply the injected ingress controller deployment yaml
kubectl get deploy -n ingress-nginx nginx-ingress-controller -oyaml | linkerd inject - | kubectl apply -f -

# Watch the pods and reload the page when the the new pod is running
kubectl get po -w -n ingress-nginx

# Observe the changes to the pods in the dashboard for the ingress-nginx namespace

# Working with Retries and Timeouts

## observe the SUCCESS statistics to HEAD /authors/{id}.json 
### it should be at about 50%
linkerd -n booksapp routes deploy/books --to svc/authors

## update the configuration to include retries
kubectl edit sp authors.booksapp.svc.cluster.local -n booksapp

# add `isRetryable: true` to the path named HEAD /authors/{id}.json
# should be the last line

## observe the statistics again and ensure that they improve
### this will take about a 60 secomds
linkerd -n booksapp routes deploy/books --to svc/authors

# timeouts

## observe the latencies
linkerd routes -n booksapp deploy/webapp --to svc/authors -t 1m -o wide

## add `timeout: 25ms` to the path named HEAD /books/{id}.json on the last line
kubectl edit sp books.booksapp.svc.cluster.local

## observe that EFFECTIVE_SUCCESS decreases
### This is visible in the dashboard as well
linkerd -n booksapp routes deploy/webapp --to svc/books -o wide

# edges review: update the nginx ingress controller to include the upstream
kubectl edit ing booksapp-ingress -n booksapp

# update the traffic deployment to use the ingress endpoint
## change line 49 to: `- ingress-nginx.ingress-nginx:80`
kubectl edit deploy/traffic -n booksapp

## under annotations add this line:
nginx.ingress.kubernetes.io/upstream-vhost: webapp.booksapp.svc.cluster.local:7000

## view the edges
linkerd edges po -n booksapp

## observe that the ingress controller is an edge to a webapp pod
