# Nginx Ingress Controller

<p align="center"> <img src="/images/nginx.png"> </p>

References: https://www.percona.com/blog/expose-databases-on-kubernetes-with-ingress

In this documentation, we want to expose `mariadb` service via `kubernetes ingress`. 

## Quick start
1. Deploy `ingress-nginx` using `helm chart`
```bash
# add helm repository
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

# update repo
helm repo update

# install the traefik chart
helm install ingress-nginx ingress-nginx/ingress-nginx
```
> Once Ingress-Nginx is installed, validate that the resources were deployed correctly, e.g. the Nginx service was created, the Nginx pod is running, etc.

2. Run mariadb pod
```bash
kubectl run mariadb --image mariadb --env="MARIADB_ROOT_PASSWORD=super-secret"
```

3. Expose mariadb via `ClusterIP`
```bash
kubectl expose pod mariadb --port 3306 --target-port 3306
```

4. Update `ingress-nginx` configuration using `values.yaml`
```bash
# Edit values.yaml
# in this config, we routes port 10001 to service 
# mariadb:3306 in namespace default

cat <<EOF > values.yaml
controller:
  service:
    enableHttp: true

tcp:
  "10001": "default/mariadb:3306"
EOF

# Update ingress-nginx
helm upgrade -f values.yaml ingress-nginx ingress-nginx/ingress-nginx
```

5. Test connection to mariadb
```bash
# Install mariadb-client
apt install mariadb-client -y

# Connect to mariadb
mariadb -h <loadbalancer-ip>  -P 10001 -u root -psuper-secret
```

## Add or remove ingress-nginx tcp port
Whenever you want to `add` or `remove` `tcp-port`, just edit the `values.yaml` and run `helm upgrade -f values.yaml ingress-nginx ingress-nginx/ingress-nginx`.

- Add new `tcp-port`
```yaml
# values.yaml
controller:
  service:
    enableHttp: true

tcp:
  "10001": "default/mariadb:3306"
  "10002": "dev/postgresql:5432"  # New entry
```

- Remove `tcp-port` `10001`
```yaml
# values.yaml
controller:
  service:
    enableHttp: true

tcp:
  "10002": "dev/postgresql:5432"
```


## Consideration
Keep in mind that `maximum` number of ports that can be exposed is `2768`. 