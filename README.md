# Heise Mastering Observability 2024 - Vector

## Colima (macOS)

- <https://github.com/abiosoft/colima>

``` shell
colima delete
colima start --arch aarch64 --vm-type=vz --vz-rosetta --runtime containerd --kubernetes --cpu 4 --memory 16 --disk 60
```

## Kube Prometheus Stack

- <https://github.com/prometheus-community/helm-charts/blob/main/charts/kube-prometheus-stack>
- <https://github.com/grafana/helm-charts/blob/main/charts/grafana>

``` shell
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

helm search repo prometheus-community/kube-prometheus-stack
helm show values prometheus-community/kube-prometheus-stack
helm show values grafana/grafana

helm upgrade kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  --install \
  --version 60.0.2 \
  --create-namespace \
  --namespace monitoring \
  --values kube-prometheus-stack/helm-values.yaml \
  --atomic

kubectl port-forward --namespace monitoring svc/kube-prometheus-stack-prometheus 8020:9090
kubectl port-forward --namespace monitoring svc/kube-prometheus-stack-alertmanager 8030:9093
kubectl port-forward --namespace monitoring svc/kube-prometheus-stack-grafana 8040:80
```

## Loki

- <https://github.com/grafana/loki/tree/main/production/helm/loki>

``` shell
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

helm search repo grafana/loki
helm show values grafana/loki

helm upgrade loki grafana/loki \
  --install \
  --version 6.6.3 \
  --create-namespace \
  --namespace monitoring \
  --values loki/helm-values.yaml \
  --atomic

kubectl port-forward --namespace monitoring svc/loki 8050:3100
```

## Vector

- <https://vector.dev/docs/setup/installation/package-managers/helm/>
- <https://github.com/vectordotdev/helm-charts/tree/develop/charts/vector>

``` shell
helm repo add vector https://helm.vector.dev
helm repo update

helm search repo vector/vector
helm show values vector/vector

helm upgrade vector vector/vector \
  --install \
  --version 0.33.0 \
  --create-namespace \
  --namespace vector \
  --values vector/agent-helm-values.yaml \
  --atomic

kubectl port-forward --namespace vector svc/vector 8060:8686

# Anzeige von Topologie und Metriken
vector top --url http://localhost:8060/graphql

# Beobachtung der Ausgabeprotokollereignisse von Quell- oder Transformationskomponenten.
vector tap --url http://localhost:8060/graphql
```

### Test

- Auslesen der Kubernetes-Logs von Vector aus Loki.

  ``` shell
  curl localhost:8050/loki/api/v1/query_range \
    --data-urlencode 'query={forwarder="vector"}' \
    | jq --color-output \
    | less
  ```

### API

- GraphQL: <http://localhost:8060/graphql>
- Playground: <http://localhost:8060/playground>

### CLI

- <https://vector.dev/docs/setup/installation/>

  ``` shell
  curl --proto '=https' --tlsv1.2 -sSfL https://sh.vector.dev | bash
  ```

### Unit Tests

``` shell
vector test vector/tests/pipeline.yaml vector/tests/test.yaml
```

### Scaffolding

``` shell
# vector generate '[sources]/[transforms]/[sinks]'
vector generate 'kubernetes_logs/remap,filter/loki'
```

### Vector Remap Language (VRL)

- Starte VRL-Shell zur Ausführung von Funktionen auf Beispiel-Input.

  ``` shell
  vector vrl --input vector/cli-vrl-example.json
  ```

- Experimentiere mit Kontext des Beispiel-Inputs.

  ``` shell
  .
  del(.file)
  .
  parse_json!(.message)
  ```
