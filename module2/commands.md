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