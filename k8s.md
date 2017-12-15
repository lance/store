```sh
oc project store
kubectl run --image=debianmaster/store-frontend:v1 store-frontend --dry-run -o yaml | istioctl kube-inject  -f - | oc apply -f -
kubectl run --image=debianmaster/store-products:v1 products --dry-run -o yaml | istioctl kube-inject  -f - | oc apply -f -
oc env deploy products MONGO_USER=app_user MONGO_PASSWORD=password MONGO_SERVER=productsdb MONGO_PORT=27017 MONGO_DB=store \
mongo_url='mongodb://app_user:password@productsdb/store'

oc create service clusterip store-frontend --tcp=80:8080 -n store 

oc project bi
kubectl run --image=debianmaster/store-recommendations:v1 store-recommendations --dry-run -o yaml | istioctl kube-inject  -f - | oc apply -f -

oc create service clusterip store-recommendations --tcp=80:8080 -n store 

oc project fflmnt
kubectl run --image=debianmaster/store-inventory:v1 store-inventory --dry-run -o yaml | istioctl kube-inject  -f - | oc apply -f -

oc create service clusterip store-inventory --tcp=80:8000 -n store 
```
