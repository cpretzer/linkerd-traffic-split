# These instructions assume that a cluster is already available to you
# Ensure that you can connect to the cluster and that kubectl 1.15+ is installed

# Windows https://storage.googleapis.com/kubernetes-release/release/v1.15.0/bin/windows/amd64/kubectl.exe
`curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.15.0/bin/windows/amd64/kubectl.exe`

# Linux
`curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl`

# Mac
`curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl`

`kubectl version --short`

# Clone the repository
`git clone https://github.com/cpretzer/linkerd-traffic-split`
`cd linkerd-traffic-split`

# Installing booksapp
`kubectl apply -f module1/booksapp.yml`

# Set the booksapp namespace as the default
`kubectl config set-context --current --namespace=booksapp`

# watch the pods start up
`kubectl get po -w`

# verify that the service has an external endpoint this may take a few minutes
`kubectl get svc -w`

# get the external URL and verify that the page loads
`kubectl get svc webapp -n booksapp -ojsonpath={.status.loadBalancer.ingress[].ip}`

# use the output from the command above in the URL of your browser
http://<output from command above>:7000

## install linkerd cli

# Linux and Mac
`curl -sL https://run.linkerd.io/install | sh`

## Windows
# Download https://github.com/linkerd/linkerd2/releases/download/stable-2.6.0/linkerd2-cli-stable-2.6.0-windows.exe
# and add it to your PATH environment variable as "linkerd.exe"

# verify the cli installation
`linkerd version`

# run a pre-check
`linkerd check --pre`

# view the linkerd control plane resources
`linkerd install`

# deploy linkerd control plane resources to the cluster
`linkerd install | kubectl apply -f -`

# check the linkerd control plane resources
`linkerd check`

# view the control plane resources in the linkerd namespace
`kubectl get po -n linkerd`
`kubectl get deploy -n linkerd`
`kubectl get svc -n linkerd`

# open the dashboard
# Note: this will run the command in the background, and will need to be killed later
`linkerd dashboard &`

# update the booksapp namespace to enable automatic proxy injection
`kubectl annotate ns booksapp linkerd.io/inject=enabled`

# restart the deployments to inject the pods
`kubectl rollout restart deploy -n booksapp`

# verify that the pods start and that there are two containers per pod
`kubectl get po -w -n booksapp`

# use linkerd tap
`linkerd tap -n booksapp deploy/webapp -o wide`
`linkerd routes -n booksapp deploy/webapp --to svc/books -o wide`
`linkerd edges -n booksapp deploy`

