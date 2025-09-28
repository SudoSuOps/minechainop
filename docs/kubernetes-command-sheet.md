# Kubernetes Command Reference

> Quick-reference commands for daily Kubernetes and K3s administration. Commands assume `kubectl` has access to the target cluster. Add `--namespace <ns>` as needed.

## Context & Authentication
- List available contexts: `kubectl config get-contexts`
- Show current context: `kubectl config current-context`
- Switch context: `kubectl config use-context <context>`
- Merge kubeconfig files: `KUBECONFIG=~/.kube/config:/path/to/extra kubectl config view --flatten > /tmp/config && mv /tmp/config ~/.kube/config`

## Cluster Health
- API components: `kubectl get componentstatuses`
- Nodes summary: `kubectl get nodes -o wide`
- Detailed node info: `kubectl describe node <node>`
- Cluster version: `kubectl version --short`

## Namespaces
- List namespaces: `kubectl get ns`
- Create namespace: `kubectl create namespace <name>`
- Delete namespace: `kubectl delete namespace <name>`

## Workloads
- Deploy manifest: `kubectl apply -f <file-or-dir>`
- Replace workload: `kubectl replace -f <file>`
- Delete workload: `kubectl delete -f <file>`
- Rolling status: `kubectl rollout status deployment/<name>`
- Rollback: `kubectl rollout undo deployment/<name>`
- Scale deployment: `kubectl scale deployment/<name> --replicas=<n>`
- List deployments with images: `kubectl get deploy -o=custom-columns=NAME:.metadata.name,IMAGES:.spec.template.spec.containers[*].image`

## Pods & Debugging
- List pods (all namespaces): `kubectl get pods -A`
- Pod details: `kubectl describe pod <pod>`
- Pod logs: `kubectl logs <pod>`
- Container logs: `kubectl logs <pod> -c <container>`
- Follow logs: `kubectl logs -f <pod>`
- Exec shell: `kubectl exec -it <pod> -- /bin/sh`
- Copy file to pod: `kubectl cp <local> <pod>:<remote>`
- Drain node: `kubectl drain <node> --ignore-daemonsets --delete-emptydir-data`
- Uncordon node: `kubectl uncordon <node>`

## Services & Networking
- List services: `kubectl get svc`
- Describe service: `kubectl describe svc <service>`
- Port-forward: `kubectl port-forward svc/<service> <local>:<remote>`
- View endpoints: `kubectl get endpoints`
- Ingress troubleshooting: `kubectl describe ingress <name>`

## Storage
- PersistentVolumeClaims: `kubectl get pvc`
- PersistentVolumes: `kubectl get pv`
- Describe PVC: `kubectl describe pvc <name>`
- Inspect StorageClass: `kubectl describe storageclass <name>`

## Helm
- List releases: `helm list -A`
- Install/upgrade: `helm upgrade --install <release> <chart> -n <ns> [--values values.yaml]`
- Show values: `helm get values <release> -n <ns>`
- Uninstall release: `helm uninstall <release> -n <ns>`

## K3s Specific
- Server service status: `sudo systemctl status k3s`
- Agent service status: `sudo systemctl status k3s-agent`
- View cluster token: `sudo cat /var/lib/rancher/k3s/server/node-token`
- Tail controller logs: `sudo journalctl -u k3s -f`
- Tail worker logs: `sudo journalctl -u k3s-agent -f`

## GPU Operations
- GPU operator pods: `kubectl get pods -n nvidia -o wide`
- Device plugin logs: `kubectl -n nvidia-device-plugin logs -l app.kubernetes.io/instance=nvdp`
- Run GPU smoke test: `kubectl run gpu-smoke --rm -it --image=nvcr.io/nvidia/cuda:12.4.1-base-ubuntu22.04 -- nvidia-smi`

## Maintenance
- Cordoning for maintenance: `kubectl cordon <node>`
- Delete completed jobs: `kubectl delete jobs --all --namespace <ns>`
- Garbage collect dangling resources: `kubectl get pods --field-selector=status.phase=Succeeded -A -o name | xargs kubectl delete`
- Backup cluster resources: `kubectl get all -A -o yaml > cluster-backup-$(date +%F).yaml`

## Troubleshooting Tips
- Watch events: `kubectl get events --sort-by=.lastTimestamp -A`
- Validate manifests: `kubectl apply --dry-run=client -f <file>`
- Resource usage: `kubectl top nodes` / `kubectl top pods -A`
- ETCD health (on controller): `sudo k3s etcd-snapshot ls` or `etcdctl endpoint health`

## Cleanup
- Delete namespace workloads: `kubectl delete all --all -n <ns>`
- Remove CRDs: `kubectl delete crd <crd-name>`
- Delete Helm release history: `helm uninstall <release> -n <ns> && kubectl delete secret -n <ns> -l owner=helm,name=<release>`
