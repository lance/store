```sh
oc project store
kubectl run --image=debianmaster/store-frontend:v1 store-frontend -l app=frontend --dry-run -o yaml | istioctl kube-inject  -f - | oc apply -f -
kubectl run --image=debianmaster/store-products:v1 products -l app=products --dry-run -o yaml | istioctl kube-inject  -f - | oc apply -f -

oc env deploy products MONGO_USER=app_user MONGO_PASSWORD=password MONGO_SERVER=productsdb MONGO_PORT=27017 MONGO_DB=store \
mongo_url='mongodb://app_user:password@productsdb/store'

oc create service clusterip frontend --tcp=80:8080 -n store 

oc project bi
kubectl run --image=debianmaster/store-recommendations:v1 recommendations -l app=recommendations --dry-run -o yaml | istioctl kube-inject  -f - | oc apply -f -

oc create service clusterip recommendations --tcp=80:8080 -n bi 

oc project fflmnt
kubectl run --image=debianmaster/store-inventory:v1 store-inventory -l app=inventory --dry-run -o yaml | istioctl kube-inject  -f - | oc apply -f -

oc create service clusterip inventory --tcp=80:8000 -n fflmnt 
```
