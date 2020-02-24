# create service profiles

# webapp
## On the browser, click "Namespaces" on the left navbar
## Click booksapp under "HTTP metrics"
## Click webapp
## Click the Route Metrics tab
linkerd profile --open-api module1/webapp.swagger webapp -n booksapp webapp > webapp-sp.yml
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

# Working with Retries and Timeouts

## observe the SUCCESS statistics to HEAD /authors/{id}.json 
### it should be at about 50%
linkerd -n booksapp routes deploy/books --to svc/authors

## update the configuration to include retries
kubectl edit serviceprofile authors.booksapp.svc.cluster.local -n booksapp

# add `isRetryable: true` to the path named HEAD /authors/{id}.json
# should be the last line

## observe the statistics again and ensure that they improve
### this will take about a 60 secomds
linkerd -n booksapp routes deploy/books --to svc/authors

# timeouts

## observe the latencies
linkerd routes -n booksapp deploy/webapp --to svc/authors -t 1m -o wide

## add `timeout: 25ms` to the path named PUT /books/{id}.json on the last line
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
