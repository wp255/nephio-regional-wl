# edge-hello-gitops

Minimal GitOps demo for an edge Kubernetes cluster using **Config Sync**.
- Central pushes YAML to GitHub, edge cluster **automatically** pulls and applies.
- Deploys a tiny HTTP echo server (hashicorp/http-echo) exposed via **NodePort 30080**.

## Structure
```
edge-hello-gitops/
├─ system/
│  └─ rootsync/
│     └─ root-sync-edge-a.yaml   # apply ONCE on the edge cluster to bootstrap
└─ clusters/
   └─ edge-a/                    # RootSync watches this directory
      ├─ namespace.yaml
      ├─ pod-hello.yaml
      └─ svc-hello-nodeport.yaml
```

## Quick start

1) **Create a GitHub repo** named `edge-hello-gitops` and push these files.

2) On the **edge cluster**, ensure Config Sync is installed, then apply the RootSync (edit the repo URL first):
```bash
kubectl apply -f system/rootsync/root-sync-edge-a.yaml
```

3) Verify:
```bash
kubectl -n config-management-system get rootsync root-sync -o jsonpath='{.status.sync.status}{"\n"}'  # expect: SYNCED
kubectl get ns edge-demo
kubectl -n edge-demo get pod hello
kubectl -n edge-demo get svc hello
```

4) Access the service:
- From outside (if you can reach a node IP):  `curl http://<EDGE_NODE_IP>:30080/`
- From inside the cluster:
  ```bash
  kubectl -n edge-demo run test --image=curlimages/curl -it --rm --         curl -sS http://hello.edge-demo.svc.cluster.local
  ```

## Notes
- If your repo is **private**, set `spec.git.auth` to `token` or `ssh` and create the referenced Secret in the
  `config-management-system` namespace.
- To change the NodePort, edit `clusters/edge-a/svc-hello-nodeport.yaml` (default is 30080).
- To switch to a Deployment instead of a single Pod, replace `pod-hello.yaml` with a Deployment manifest.
