# Inventaire — Stage 00

## Nœuds

| Nœud | Rôle | IP | Labels | Taints |
|---|---|---|---|---|
| lab404-control-plane | control-plane | 172.28.0.3 | node-role.kubernetes.io/control-plane | node-role.kubernetes.io/control-plane:NoSchedule |
| lab404-worker | worker | 172.28.0.4 | | |
| lab404-worker2 | worker | 172.28.0.2 | lab404.io/workload=stateful | workload=stateful:NoSchedule |

Kubernetes v1.35.0, containerd 2.2.0, Debian 12.

## Namespaces

default, envoy-gateway-system, kube-node-lease, kube-public, kube-system, lab404, local-path-storage, metallb-system, observability

## Workloads lab404

| Deployment | Image | Pod IP | Nœud |
|---|---|---|---|
| web | ghcr.io/estelle-tamalet/lab404-web:latest | 10.244.1.8 | lab404-worker |
| votes | ghcr.io/estelle-tamalet/lab404-votes:latest | 10.244.1.7 | lab404-worker |
| valkey | valkey/valkey:8.1-alpine | 10.244.2.2 | lab404-worker2 |
| nginx | nginx:latest | 10.244.1.6 | lab404-worker |

## Workloads plateforme

| Namespace | Workload | Type | Image | Nœud(s) |
|---|---|---|---|---|
| envoy-gateway-system | envoy-gateway | Deployment | envoyproxy/gateway:v1.8.3 | worker |
| envoy-gateway-system | envoy-lab404-lab404-063cf357 | Deployment | envoyproxy/envoy:distroless-v1.38.3 | worker |
| observability | otel-collector | Deployment | otel/opentelemetry-collector-k8s:0.156.0 | worker |
| metallb-system | metallb-controller | Deployment | metallb/controller:v0.16.1 | worker |
| metallb-system | metallb-speaker | DaemonSet | metallb/speaker:v0.16.1 | control-plane, worker |
| metallb-system | metallb-frr-k8s | DaemonSet | metallb/frr-k8s:v0.0.25, frr:10.4.3 | control-plane, worker |
| kube-system | coredns | Deployment | coredns:v1.13.1 | control-plane |
| kube-system | kindnet, kube-proxy | DaemonSet | kindnetd, kube-proxy:v1.35.0 | tous |
| local-path-storage | local-path-provisioner | Deployment | local-path-provisioner | control-plane |

## Services

| Namespace | Service | Type | Cluster IP | IP externe | Ports |
|---|---|---|---|---|---|
| envoy-gateway-system | envoy-lab404-lab404-063cf357 | LoadBalancer | 10.96.100.238 | 172.28.255.201 | 80:30920 |
| lab404 | nginx | LoadBalancer | 10.96.98.237 | 172.28.255.200 | 80:31154 |
| lab404 | web | ClusterIP | 10.96.91.96 | | 8080 |
| lab404 | votes | ClusterIP | 10.96.98.87 | | 8081 |
| lab404 | valkey | ClusterIP | 10.96.98.51 | | 6379 |
| observability | otel-collector | ClusterIP | 10.96.36.80 | | 4317, 4318 |
| envoy-gateway-system | envoy-gateway | ClusterIP | 10.96.175.169 | | 18000-18002, 19001, 9443 |
| kube-system | kube-dns | ClusterIP | 10.96.0.10 | | 53, 9153 |
| default | kubernetes | ClusterIP | 10.96.0.1 | | 443 |

## Gateway API

| Ressource | Nom | Détail |
|---|---|---|
| GatewayClass | lab404 | gateway.envoyproxy.io/gatewayclass-controller |
| Gateway | lab404/lab404 | 172.28.255.201 |
| HTTPRoute | lab404/web | PathPrefix / → web:8080 |

## Configuration des flux

| Workload | Variable | Valeur |
|---|---|---|
| web | VOTES_URL | http://votes:8081 |
| web | OTEL_EXPORTER_OTLP_ENDPOINT | http://otel-collector.observability.svc.cluster.local:4318 |
| votes | VALKEY_ADDR | valkey:6379 |
| votes | VALKEY_PASSWORD | secret valkey/password |
| votes | OTEL_EXPORTER_OTLP_ENDPOINT | http://otel-collector.observability.svc.cluster.local:4318 |
| votes | OTEL_EXPORTER_OTLP_INSECURE | true |
| valkey | VALKEY_PASSWORD | secret valkey/password |

## Secrets

| Namespace | Secret | Type |
|---|---|---|
| lab404 | valkey | Opaque |
| kube-system | bootstrap-token-abcdef | bootstrap.kubernetes.io/token |
| envoy-gateway-system | envoy, envoy-gateway, envoy-rate-limit | kubernetes.io/tls |
| envoy-gateway-system | envoy-oidc-hmac | Opaque |
| metallb-system | metallb-memberlist, metallb-webhook-cert, frr-k8s-webhook-server-cert | Opaque |

## Identités

| Namespace | ServiceAccount | Pods | Bindings |
|---|---|---|---|
| lab404 | default | nginx, valkey, votes, web | aucun |

## Adresses exposées

| Adresse | Cible |
|---|---|
| 172.28.255.201:80 | Envoy Gateway |
| 172.28.255.200:80 | nginx |
| 172.28.0.2-4:30920 | NodePort Envoy |
| 172.28.0.2-4:31154 | NodePort nginx |
| 127.0.0.1:6443 | API Kubernetes |