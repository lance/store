|Microservice|Namespace|Features Covered|Contents|
|------------|--|----------------|-|
|store-frontend|store|dockerfile build|angular app|
|store-inventory|fflmnt|multi stage builds, FROM scratch,gitlab-ci,distributed databases|golang api,cockroach db, init containers|
|store-products|store|s2i builds, linking services|nodejs api, mongodb,traffic split|
|store-recommendations|store|s2i builds|nodejs api, mongodb,traffic split|
|All|all|network policies, microservice ops||

## Bootstrap
```sh
oc new-project fflmnt
oc new-project store
oc new-project bi
```
## Store frontend
```sh
cd ~/store-frontend
docker build -t "debianmaster/store-fe:v1" .
docker push debianmaster/store-fe:v1
```

## Store products
```sh
oc project store
```
### DB
```sh
oc new-app mongodb -l app=mongodb --name=productsdb \
  -e MONGODB_ADMIN_PASSWORD=password  -e MONGODB_USER=app_user \
  -e MONGODB_DATABASE=store  -e MONGODB_PASSWORD=password
  
```
### API
```sh
cd ~/store-products

git checkout master
s2i build . node:8-slim debianmaster/store-products:v1
docker push debianmaster/store-products:v1

git checkout recommendations
s2i build . node:8-slim debianmaster/store-products:recommendations
docker push debianmaster/store-products:recommendations

oc new-app debianmaster/store-products:v1 --name=products

oc env dc products MONGO_USER=app_user MONGO_PASSWORD=password MONGO_SERVER=productsdb MONGO_PORT=27017 MONGO_DB=store \
mongo_url='mongodb://app_user:password@productsdb/store'
```

## Store inventory
```sh
oc project fflmnt
```
### DB
```sh
oc adm policy add-scc-to-user anyuid -z default
oc apply -f https://raw.githubusercontent.com/cockroachdb/cockroach/master/cloud/kubernetes/cockroachdb-statefulset.yaml
oc apply -f https://raw.githubusercontent.com/cockroachdb/cockroach/master/cloud/kubernetes/cluster-init.yaml
oc expose svc  cockroachdb-public --port=8080 --name=r1
oc scale statefulset cockroachdb --replicas=4
oc run cockroachdb -it --image=cockroachdb/cockroach --rm --restart=Never     -- sql --insecure --host=cockroachdb-public
```
```sql
create database store;
use store;
create table inventory (id int,product_id varchar(30),product_cost int,product_availabilty int,product_subcat int);
insert into inventory values (1,'cable_1',10,200,1);
```
### API
```sh
cd ~/store-inventory
docker build -t debianmaster/store-inventory:v1 -f Dockerfile.scratch .
docker push debianmaster/store-inventory:v1
oc new-app debianmaster/store-inventory:v1 --name=inventory \
-e sql_string=postgresql://root@cockroachdb-public:26257/store?sslmode=disable
oc expose svc inventory
```

## Store Recommendations
```sh
oc project bi
```
```sh
cd ~/store-recommendations
docker build -t debianmaster/store-recommendations:v1 .
docker push debianmaster/store-recommendations:v1
```

## Istio
### Permissions
```sh
oc adm policy add-scc-to-user anyuid -z istio-ingress-service-account -n istio-system
oc adm policy add-scc-to-user anyuid -z default -n istio-system
oc adm policy add-scc-to-user privileged -z default -n store
oc adm policy add-scc-to-user privileged -z default -n fflmnt
oc adm policy add-scc-to-user privileged -z default -n bi
```
```sh Istio setup
oc apply -f ~/istio-workshop/istio/install/istio-0.3.0/install/kubernetes/istio.yaml
```



