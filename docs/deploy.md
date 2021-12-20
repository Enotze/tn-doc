## Настройка деплоя ##

```bash
# storage для хранения вендоров
gsutil mb -p $CLOUDSDK_CORE_PROJECT -l EUROPE-WEST1 gs://${CLOUDSDK_CORE_PROJECT}-artifacts

# secrets для sql proxy
kubectl create secret generic cloudsql-instance-credentials \
    --from-file=cloudsqlproxy-manager.json=cloud-proxy-sql-tech-control-178408-23d67c28e9fa.json

# Пароль для подключения к postgress
kubectl create secret generic postgress-staging --from-literal=password=PASSWORD

# JWT secret (artisan jwt:secret)
kubectl create secret generic jwt --from-literal=secret=SECRET
```
