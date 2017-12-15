```sh
oc project store
kubectl run --image=debianmaster/store-frontend:v1 store-frontend --dry-run -o yaml | istioctl kube-inject  -f - | oc apply -f -

oc create service clusterip store-frontend --tcp=80:8080 -n store 

oc project bi
kubectl run --image=debianmaster/store-recommendations:v1 store-frontend --dry-run -o yaml | istioctl kube-inject  -f - | oc apply -f -

oc create service clusterip store-recommendations --tcp=80:8080 -n store 

oc project fflmnt
kubectl run --image=debianmaster/store-inventory:v1 store-frontend --dry-run -o yaml | istioctl kube-inject  -f - | oc apply -f -

oc create service clusterip store-inventory --tcp=80:8080 -n store 
```
