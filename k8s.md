```sh
oc project store
kubectl run --image=debianmaster/store-frontend:v1 store-frontend --dry-run -o yaml | istioctl kube-inject  -f - | oc apply -f -

oc project bi
kubectl run --image=debianmaster/store-recommendations:v1 store-frontend --dry-run -o yaml | istioctl kube-inject  -f - | oc apply -f -

oc project fflmnt
kubectl run --image=debianmaster/store-inventory:v1 store-frontend --dry-run -o yaml | istioctl kube-inject  -f - | oc apply -f -
```
