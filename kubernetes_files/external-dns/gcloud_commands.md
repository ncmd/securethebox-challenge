# Notes
**check context**
kubectl config get-contexts
kubectl config use-context docker-for-desktop

**enable dns api**
https://console.cloud.google.com/flows/enableapi?apiid=dns&_ga=2.33202941.-258188431.1550904141

**enable dns api**


**gke**
```
gcloud auth login
gcloud config set project "securethebox"
gcloud config set compute/region "us-west1"
gcloud config set compute/zone "us-west1-a"
```
**Create a GKE cluster.**
```
gcloud container clusters create "external-dns" \
    --num-nodes 1 \
    --scopes "https://www.googleapis.com/auth/ndev.clouddns.readwrite"
```
**Create a DNS zone which will contain the managed DNS records.**
```
gcloud dns managed-zones create "external-dns-test-securethebox-us" \
    --dns-name "external-dns-test.securethebox.us." \
    --description "Automatically managed zone by kubernetes.io/external-dns"
```
**Make a note of the nameservers that were assigned to your new zone.**
```
gcloud dns record-sets list \
    --zone "external-dns-test-securethebox-us" \
    --name "external-dns-test.securethebox.us." \
    --type NS
```
kubectl create clusterrolebinding your-user-cluster-admin-binding --clusterrole=cluster-admin --user=cchong.vise@gmail.com

**Tell the parent zone where to find the DNS records for this zone by adding the corresponding NS records there. Assuming the parent zone is "gcp-zalan-do" and the domain is "gcp.zalan.do" and that it's also hosted at Google we would do the following.**
```
gcloud dns record-sets transaction start --zone "securethebox-us"
gcloud dns record-sets transaction add ns-cloud-d{1..4}.googledomains.com. \
    --name "external-dns-test.securethebox.us." --ttl 300 --type NS --zone "securethebox-us"
gcloud dns record-sets transaction execute --zone "securethebox-us"
```
# Deploy RBAC ExternalDNS and nginx (wait 2 minutes)
```
gcloud container clusters get-credentials "external-dns"
kubectl apply -f rbac-external-dns.yml
kubectl apply -f verify.yml
```



## Check DNS records created in CLOUD DNS
```
gcloud dns record-sets list \
    --zone "external-dns-test-securethebox-us" \
    --name "nginx.external-dns-test.securethebox.us."
```
## Check domain is resolvable
```
dig +short @ns-cloud-d1.googledomains.com. nginx.external-dns-test.securethebox.us.
```
## Check nginx content is downloadable
```
curl http://nginx.external-dns-test.securethebox.us
```
# Create Ingress (wait 2 minutes)
```
kubectl apply -f ingress.yml 
```
# 
```
gcloud dns record-sets list \
    --zone "external-dns-test-securethebox-us" \
    --name "via-ingress.external-dns-test.securethebox.us."
```

#
dig +short @ns-cloud-d1.googledomains.com. via-ingress.external-dns-test.securethebox.us.

curl via-ingress.external-dns-test.securethebox.us

**connect to cluster**
```
gcloud container clusters get-credentials stb-cluster-1 --zone us-central1-a --project securethebox
```

**check dns records**
```
gcloud dns record-sets list \
    --zone "external-dns-test-securethebox-us" \
    --name "nginx.external-dns-test.securethebox.us."
```

1 minute after deployment for dns record to be created in CLOUDDNS
2 minutes after deployment for dns record to be applied
~4 minutes for nginx service to load

**delete cluster**
```
gcloud container clusters delete "external-dns"
```