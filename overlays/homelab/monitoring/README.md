# Monitoring Overlay (HA Profile)

This overlay runs Prometheus, Alertmanager, Grafana, and Loki with a high-availability baseline for the homelab cluster.

## HA settings

- Prometheus: `replicas=2`, anti-affinity, topology spread, PDB.
- Query path: Thanos Sidecar on each Prometheus pod + `thanos-query` fan-in service with replica deduplication.
- Alertmanager: `replicas=3`, anti-affinity, topology spread, PDB.
- Prometheus Operator: `replicas=2` via overlay patch, anti-affinity, topology spread, PDB.
- kube-state-metrics: `replicas=2`, anti-affinity, topology spread, PDB.
- Grafana: `replicas=2`, anti-affinity, topology spread, PDB.
- Loki: `replicas=2`, memberlist ring, replication factor `2`, PDB.

## Important note on Grafana persistence

Grafana is configured as stateless (`emptyDir` for `/var/lib/grafana`) to allow multi-replica scheduling with the current storage profile.
Dashboards and datasources are provisioned from ConfigMaps.
Any UI-local changes inside Grafana are ephemeral and must be captured in Git.

## Important note on Loki durability

Loki currently uses filesystem-backed storage on per-pod PVCs with replication factor `2`.
This improves availability for single-pod failures, but the long-term target for stronger durability is object storage-backed Loki.

## Query HA behavior

- Grafana uses `thanos-query` as Prometheus datasource endpoint.
- `thanos-query` discovers Prometheus sidecars via DNS SRV against `monitoring-kube-prometheus-thanos-discovery`.
- Replica deduplication uses the label `prometheus_replica`.
- This avoids flapping/no-data results when metrics exist only on one Prometheus replica TSDB.
