# Host Down Runbook

## Alert

`HostDown`

## Severity

Critical

## Meaning

Prometheus cannot scrape `node_exporter` from the target host for more than 2 minutes.

This may indicate that the host is down, unreachable, blocked by firewall, overloaded, or that the `node_exporter` service or pod is not working correctly.

---

## Initial Triage

### 1. Check host reachability

```bash
ping <host-ip>
```

Example:

```bash
ping 10.10.2.71
```

### 2. Check node_exporter port

```bash
nc -vz <host-ip> 9100
```

Example:

```bash
nc -vz 10.10.2.71 9100
```

### 3. Check SSH access

```bash
ssh <user>@<host-ip>
```

Example:

```bash
ssh k8@10.10.2.71
```

---

## Kubernetes Node Check

If the affected host is a Kubernetes node, check node status:

```bash
kubectl get nodes -o wide
```

Describe the affected node:

```bash
kubectl describe node <node-name>
```

Check pods running on the affected node:

```bash
kubectl get pods -A -o wide | grep <node-name>
```

---

## node_exporter Check

Check node_exporter pods:

```bash
kubectl get pods -n monitoring -o wide | grep node-exporter
```

Describe the affected node_exporter pod:

```bash
kubectl describe pod <node-exporter-pod> -n monitoring
```

Check node_exporter service:

```bash
kubectl get svc -n monitoring | grep node-exporter
```

---

## Firewall Check

If the host is reachable but Prometheus cannot scrape port `9100`, check firewall rules.

On the affected node:

```bash
sudo ufw status
```

Allow Prometheus/node_exporter scrape traffic if required:

```bash
sudo ufw allow 9100/tcp
sudo ufw reload
```

For stricter access, allow only from Prometheus node or monitoring subnet.

Example:

```bash
sudo ufw allow from <prometheus-node-ip> to any port 9100 proto tcp
```

---

## Common Causes

| Cause | Description |
| --- | --- |
| Host down | VM or physical host is powered off |
| Network issue | Routing, VLAN, firewall, or pfSense rule is blocking access |
| node_exporter down | node_exporter pod/service is not running |
| Firewall block | UFW or network firewall blocks port 9100 |
| Kubernetes node issue | Node is NotReady or kubelet/container runtime is unhealthy |
| Prometheus target issue | ServiceMonitor or scrape target is misconfigured |

---

## Resolution Steps

1. Restore host power or VM availability.
2. Restore network connectivity.
3. Allow Prometheus scrape access to port `9100`.
4. Restart node_exporter if required.
5. Check kubelet and container runtime if the node is unhealthy.
6. Confirm Prometheus target returns to `UP`.
7. Confirm Alertmanager alert is resolved.

---

## Validation After Fix

Check Prometheus target:

```promql
up{job=~".*node-exporter.*"}
```

Expected value:

```text
1
```

Check Kubernetes node:

```bash
kubectl get nodes -o wide
```

Check Alertmanager:

```text
Alert should move from firing to resolved.
```

---

## Post-Incident Follow-Up

- Document root cause.
- Add preventive action.
- Update firewall or routing documentation if required.
- Tune alert threshold if it was a false positive.
- Update monitoring inventory.
- Add GitHub Issue or incident ticket for follow-up action.