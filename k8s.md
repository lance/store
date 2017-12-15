```sh
kubectl run --image=debianmaster/store-frontend:v1 store-frontend --dry-run -o yaml | istioctl kube-inject  -f - | oc apply -f -
```
