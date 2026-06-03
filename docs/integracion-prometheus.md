# Integracion de Pulso con Prometheus y Grafana

Esta guia explica como configurar Pulso para exponer metricas en formato Prometheus, configurar Prometheus para hacer scraping y crear dashboards en Grafana.

## Requisitos previos

- Pulso instalado y configurado
- Prometheus version 2.x o superior
- Grafana version 8.x o superior

## 1. Activar formato Prometheus en Pulso

Configurar pulso.toml:

[output]
format = "prometheus"
port = 9090
endpoint = "/metrics"

Verificar:

curl http://localhost:9090/metrics

## 2. Configurar Prometheus scraping

Ejemplo de prometheus.yml:

global:
  scrape_interval: 15s

scrape_configs:
  - job_name: pulso
    scrape_interval: 10s
    metrics_path: /metrics
    static_configs:
      - targets: [localhost:9090]
        labels:
          service: pulso
          environment: production

Verificar:

promtool check config prometheus.yml
curl http://localhost:9090/api/v1/targets

## 3. Metricas expuestas

| Nombre | Tipo | Descripcion |
|--------|------|-------------|
| pulso_requests_total | Counter | Total de requests procesadas |
| pulso_request_duration_seconds | Histogram | Duracion de requests |
| pulso_errors_total | Counter | Total de errores |
| pulso_active_connections | Gauge | Conexiones activas |
| pulso_memory_usage_bytes | Gauge | Memoria utilizada |
| pulso_cpu_usage_seconds_total | Counter | Tiempo de CPU |
| pulso_up | Gauge | Estado del servicio |

Queries PromQL:

rate(pulso_requests_total[5m])
histogram_quantile(0.95, rate(pulso_request_duration_seconds_bucket[5m]))
rate(pulso_errors_total[5m])
pulso_memory_usage_bytes{type="heap"}

## 4. Dashboard de Grafana

Paneles recomendados:

1. Tasa de requests: rate(pulso_requests_total[5m])
2. Latencia P95: histogram_quantile(0.95, rate(pulso_request_duration_seconds_bucket[5m]))
3. Tasa de errores: rate(pulso_errors_total[5m])
4. Conexiones activas: pulso_active_connections
5. Uso de memoria: pulso_memory_usage_bytes / 1024 / 1024
6. Uso de CPU: rate(pulso_cpu_usage_seconds_total[5m])
7. Top endpoints: topk(10, sum by (endpoint) (rate(pulso_requests_total[1h])))

## 5. Verificar integracion

curl -s http://localhost:9090/metrics | head -20
curl -s http://localhost:9090/api/v1/targets | jq .data.activeTargets[]
curl -X GET http://admin:admin@localhost:3000/api/datasources

## 6. Alertas

alerts.yml:

groups:
  - name: pulso_alerts
    interval: 30s
    rules:
      - alert: PulsoDown
        expr: pulso_up == 0
        for: 1m
        labels:
          severity: critical
      - alert: HighErrorRate
        expr: rate(pulso_errors_total[5m]) > 0.1
        for: 2m
        labels:
          severity: warning

## Troubleshooting

Verificar Pulso: ps aux | grep pulso
Verificar puerto: netstat -tlnp | grep 9090
Verificar datasource en Grafana

## Referencias

https://prometheus.io/docs/
https://grafana.com/docs/grafana/latest/dashboards/
environment: production

Verificar:

promtool check config prometheus.yml
curl http://localhost:9090/api/v1/targets

## 4. Dashboard de Grafana

Paneles recomendados:

1. Tasa de requests: rate(pulso_requests_total[5m])
2. Latencia P95: histogram_quantile(0.95, rate(pulso_request_duration_seconds_bucket[5m]))
3. Tasa de errores: rate(pulso_errors_total[5m])
4. Conexiones activas: pulso_active_connections
5. Uso de memoria: pulso_memory_usage_bytes / 1024 / 1024
6. Uso de CPU: rate(pulso_cpu_usage_seconds_total[5m])
7. Top endpoints: topk(10, sum by (endpoint) (rate(pulso_requests_total[1h])))

## 5. Verificar integracion

curl -s http://localhost:9090/metrics | head -20
curl -s http://localhost:9090/api/v1/targets | jq '.data.activeTargets[]'
curl -X GET http://admin:admin@localhost:3000/api/datasources

## 6. Alertas

Crear alerts.yml:

groups:
  - name: pulso_alerts
    interval: 30s
    rules:
      - alert: PulsoDown
        expr: pulso_up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Pulso service is down"

      - alert: HighErrorRate
        expr: rate(pulso_errors_total[5m]) > 0.1
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "High error rate detected"

## Troubleshooting

Prometheus no puede scrapear:
- Verificar Pulso corriendo: ps aux | grep pulso
- Verificar puerto: netstat -tlnp | grep 9090

Metricas no aparecen en Grafana:
- Verificar datasource configurado
- Probar query en Explore

## Referencias

- Prometheus documentation: https://prometheus.io/docs/
- PromQL basics: https://prometheus.io/docs/prometheus/latest/querying/basics/
- Grafana dashboards: https://grafana.com/docs/grafana/latest/dashboards/

## Proximos pasos

1. Configurar retention de metricas
2. Implementar Alertmanager
3. Crear SLIs y SLOs
4. Configurar recording rules
