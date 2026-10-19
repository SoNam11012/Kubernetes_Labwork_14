# Kubernetes_Labwork_14
This repo contains the files  that were used during lab work 14
# CTE412 Practical 15 — Campus Notice Board on Kubernetes

Student: Sonam Choejur | Student number: 02230148

Rolling, Recreate and blue/green releases of the Notice Board API on a kind cluster. A monitor Pod probes the service every 0.5 s so downtime is measured, not assumed.

## Versions
OS: Ubuntu 24.04 LTS | Docker: 27.3.1 | kind: v0.24.0 | kubectl: v1.31.0 | Kubernetes: v1.31.0 | Git: 2.43.0 | App: Python 3.12 / python:3.12-alpine

## Layout
cluster/ · app/ · manifests/{base,rolling,bluegreen}/ · scripts/summarise.sh · evidence/ · report/p06-report.md

## Rebuild final state (Task 8)
```bash


# Cluster, namespace, quota, limit range
kind create cluster --config cluster/kind-cluster.yaml
kubectl config use-context kind-cte412-$SID
kubectl apply -f manifests/base/namespace.yaml -f manifests/base/quota.yaml -f manifests/base/limitrange.yaml
kubectl config set-context --current --namespace=nb-$SID

# Images (local build, no registry)
for v in 1.0.0 1.1.0 1.2.0-broken; do
  docker build -t notice-board:$SID-$v --build-arg VERSION=$v --build-arg STUDENT=$SID app/
  kind load docker-image notice-board:$SID-$v --name cte412-$SID
done

# Blue/green + flag + monitor
kubectl apply -f manifests/bluegreen/
kubectl rollout status deployment/notice-board-blue
kubectl rollout status deployment/notice-board-green
kubectl apply -f manifests/base/monitor.yaml

# Verify
curl -s http://127.0.0.1:30090/version
kubectl logs monitor > /tmp/m.log && scripts/summarise.sh /tmp/m.log
```

## Clean-up
```bash
kubectl logs monitor > evidence/t9-monitor-full.log
kubectl config set-context --current --namespace=default
kind delete cluster --name cte412-$SID
git add -A && git commit -m "Task 9: rebuild proof and clean-up"
```

## Notes
- All changes are declarative; the only imperative change is `kubectl rollout undo` in Task 6.
- Evidence is saved in `evidence/` as `tN-<description>.txt` (monitor logs as `.log`) and shows `<sid>`.
- No registry is used: images are built locally and loaded with `kind load`, and all manifests set `imagePullPolicy: Never`.
- Report: `report/p06-report.md`
