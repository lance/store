|Microservice|Features Covered|Contents|
|------------|----------------|-|
|store-frontend|dockerfile build|angular app|
|store-inventory|multi stage builds, FROM scratch,gitlab-ci,distributed databases|golang api,cockroach db, init containers|
|store-products|s2i builds|nodejs api, mongodb,traffic split|


## Store frontend
```sh
cd ~/store-frontend
docker build -t "debianmaster/store-fe:v1" .
docker push debianmaster/store-fe:v1
```

## Store products
```sh
cd ~/store-products
s2i build . node:8-slim debianmaster/store-products:v1
docker push debianmaster/store-products:v1
```

## Store inventory
```sh
cd ~/store-inventory
docker build -t debianmaster/store-inventory:v1 -f Dockerfile.scratch .
docker push debianmaster/store-inventory:v1
```
