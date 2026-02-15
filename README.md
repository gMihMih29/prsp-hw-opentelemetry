# ДЗ Opentelemetry. Гетманов Михаил

```bash
kubectl delete namespace observability
kubectl apply -f namespace.yaml
```

```bash
helm repo add jetstack https://charts.jetstack.io
helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

## cert manager
```bash
helm upgrade --install cert-manager jetstack/cert-manager \
  --namespace cert-manager --create-namespace \
  --set installCRDs=true
```

## Zipkin
```bash
kubectl apply -f zipkin.yaml
kubectl -n observability port-forward svc/zipkin 9411:9411
```
[Страница](http://localhost:9411/zipkin/)

## LOKI
```bash
helm upgrade --install loki grafana/loki -n observability --create-namespace -f loki-values.yaml
```
<!-- http://loki-gateway.observability.svc.cluster.local/ -->

## Prometheus
```bash
kubectl apply -f prometheus.yaml
```

## Grafana
```bash
# kubectl apply -f grafana.yaml
helm upgrade --install grafana grafana/grafana --namespace observability -f values-grafana.yaml
export POD_NAME=$(kubectl get pods --namespace observability -l "app.kubernetes.io/name=grafana,app.kubernetes.io/instance=grafana" -o jsonpath="{.items[0].metadata.name}")
kubectl --namespace observability port-forward $POD_NAME 3000
```

## otel
```bash
helm upgrade --install opentelemetry-operator open-telemetry/opentelemetry-operator -n observability --create-namespace --set admissionWebhooks.certManager.enabled=true
kubectl --namespace observability get pods -l "app.kubernetes.io/instance=opentelemetry-operator"
kubectl get crd | grep opentelemetry
```

```bash
kubectl apply -f otel.yaml
kubectl apply -f otel-instrumentation.yaml
```

## Проверка
[Посмотреть трейс в графане](http://127.0.0.1:3000/explore?schemaVersion=1&panes=%7B%2262k%22:%7B%22datasource%22:%22zipkin%22,%22queries%22:%5B%7B%22refId%22:%22A%22,%22datasource%22:%7B%22type%22:%22zipkin%22,%22uid%22:%22zipkin%22%7D,%22queryType%22:%22traceID%22,%22query%22:%226f77a247db88714214665b4045636ec6%22%7D%5D,%22range%22:%7B%22from%22:%22now-1h%22,%22to%22:%22now%22%7D,%22panelsState%22:%7B%22trace%22:%7B%22spanFilters%22:%7B%22spanNameOperator%22:%22%3D%22,%22serviceNameOperator%22:%22%3D%22,%22fromOperator%22:%22%3E%22,%22toOperator%22:%22%3C%22,%22tags%22:%5B%7B%22id%22:%226a1d3767-f95%22,%22operator%22:%22%3D%22%7D%5D%7D%7D%7D,%22compact%22:false%7D%7D&orgId=1)
[Посмотреть логи в графане](http://127.0.0.1:3000/explore?schemaVersion=1&panes=%7B%2262k%22:%7B%22datasource%22:%22loki%22,%22queries%22:%5B%7B%22refId%22:%22A%22,%22expr%22:%22%7Bservice_name%3D%5C%22muffin-wallet%5C%22%7D%20%7C%3D%20%60%60%22,%22queryType%22:%22range%22,%22datasource%22:%7B%22type%22:%22loki%22,%22uid%22:%22loki%22%7D,%22editorMode%22:%22builder%22,%22direction%22:%22backward%22%7D%5D,%22range%22:%7B%22from%22:%22now-1h%22,%22to%22:%22now%22%7D,%22panelsState%22:%7B%22trace%22:%7B%22spanFilters%22:%7B%22spanNameOperator%22:%22%3D%22,%22serviceNameOperator%22:%22%3D%22,%22fromOperator%22:%22%3E%22,%22toOperator%22:%22%3C%22,%22tags%22:%5B%7B%22id%22:%226a1d3767-f95%22,%22operator%22:%22%3D%22%7D%5D%7D%7D%7D,%22compact%22:false%7D%7D&orgId=1)

Графики:
- [кол-во запросов в секунду по методам](http://127.0.0.1:3000/explore?schemaVersion=1&panes=%7B%2262k%22:%7B%22datasource%22:%22prometheus%22,%22queries%22:%5B%7B%22refId%22:%22A%22,%22expr%22:%22sum%20by%20%28method%29%20%28rate%28http_server_requests_seconds_count%5B1m%5D%29%29%22,%22range%22:true,%22datasource%22:%7B%22type%22:%22prometheus%22,%22uid%22:%22prometheus%22%7D,%22editorMode%22:%22code%22,%22legendFormat%22:%22__auto%22%7D%5D,%22range%22:%7B%22from%22:%22now-1h%22,%22to%22:%22now%22%7D,%22panelsState%22:%7B%22trace%22:%7B%22spanFilters%22:%7B%22spanNameOperator%22:%22%3D%22,%22serviceNameOperator%22:%22%3D%22,%22fromOperator%22:%22%3E%22,%22toOperator%22:%22%3C%22,%22tags%22:%5B%7B%22id%22:%226a1d3767-f95%22,%22operator%22:%22%3D%22%7D%5D%7D%7D%7D,%22compact%22:false%7D%7D&orgId=1)

# debug
```bash
kubectl get pods -n observability
kubectl get pods -n cert-manager
kubectl get pods
kubectl get pods -A
```

```bash
kubectl -n observability get svc opentelemetry-operator-webhook
kubectl -n observability get endpoints opentelemetry-operator-webhook
kubectl get pods -A | grep opentelemetry-operator
```

```bash
kubectl get mutatingwebhookconfigurations | grep opentelemetry
kubectl get validatingwebhookconfigurations | grep opentelemetry
```

```bash
kubectl delete mutatingwebhookconfigurations opentelemetry-operator-mutating --ignore-not-found
kubectl delete mutatingwebhookconfigurations opentelemetry-operator-validating --ignore-not-found
```