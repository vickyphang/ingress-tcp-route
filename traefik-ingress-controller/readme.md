# Traefik Kubernetes Ingress

<p align="center"> <img src="/images/traefik.png"> </p>

References: https://doc.traefik.io/traefik/routing/providers/kubernetes-crd/

In this documentation, we want to expose `mariadb` service via `kubernetes ingress`. 

## Quick start
1. Deploy `traefik` using `helm chart`
```bash
# add helm repository
helm repo add traefik https://helm.traefik.io/traefik

# update repo
helm repo update

# install the traefik chart
helm install traefik traefik/traefik
```
> Once Traefik is installed, validate that the resources were deployed correctly, e.g. the Traefik service was created, the Traefik pod is running, etc.

2. Run mariadb pod
```bash
kubectl run mariadb --image mariadb --env="MARIADB_ROOT_PASSWORD=super-secret"
```

3. Expose mariadb via `ClusterIP`
```bash
kubectl expose pod mariadb --port 3306 --target-port 3306
```

4. Create ingress route
```bash
# Edit this ingress route
cat <<EOF > ingress-route.yaml
apiVersion: traefik.io/v1alpha1
kind: IngressRouteTCP
metadata:
  name: ingress-mariadb
  namespace: default
spec:
  entryPoints:
    - ep-1
  routes:
    - match: HostSNI(`*`)
      services:
        - name: mariadb
          port: 3306
EOF

# Apply
kubectl apply -f ingress-route.yaml
```
> to validate that the resources were deployed correctly, use `kubectl get ingressroutetcp -A`


5. Update `traefik` configuration using `values.yaml`
```bash
# Edit values.yaml
# in this yaml, we use ep-1 as the port-name or entrypoint.
# it exposes port 10001 with protocol TCP
cat <<EOF > values.yaml
ports:
  ep-1:
    port: 10001
    expose:
      default: true
    exposedPort: 10001
    protocol: TCP
EOF

# Update traefik
helm upgrade --values values.yaml traefik traefik/traefik
```

6. Test connection to mariadb
```bash
# Install mariadb-client
apt install mariadb-client -y

# Connect to mariadb
mariadb -h <loadbalancer-ip>  -P 10001 -u root -psuper-secret
```