```bash
kubectl config view
```

```bash
kubectl create ns annuaire
kubectl config set-context --current --namespace annuaire
```

```bash
kubectl -n annuaire
```

```bash
kubectl -n annuaire get pods
kubectl -n annuaire get svc
kubectl -n annuaire get ing
kubectl -n annuaire get pvc
kubectl -n annuaire get deploy
```

```bash
kubectl apply -f _db/ann-mariadb-pvc.yaml
kubectl apply -f _db/mariadb.yaml
kubectl apply -f _db/
```

```bash
#
kubectl apply -f _cm/
#
kubectl delete -f _fastapi/
kubectl apply -f _fastapi/
#
kubectl -n annuaire describe pods py-ss-0
kubectl -n annuaire logs -f py-ss-0
#
kubectl -n annuaire rollout restart sts/py-ss
kubectl -n annuaire rollout status sts/py-ss
```

```bash
kubectl apply -f _ingress/
```

## Monitoring

```bash
kubectl -n annuaire top pods
kubectl top pods
```
