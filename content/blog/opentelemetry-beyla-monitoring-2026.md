---
title: "OpenTelemetry + Beyla: monitoring bez změny kódu v roce 2026"
description: "Jak nasadit kompletní observability stack v roce 2026 s OpenTelemetry Collector, Grafana Beyla eBPF auto-instrumentací a AI-assisted monitoringem na Kubernetes a OpenShift."
date: 2026-04-18
lastmod: 2026-04-18
keywords: ["OpenTelemetry 2026", "Grafana Beyla eBPF", "observability Kubernetes 2026", "monitoring bez změny kódu", "eBPF monitoring aplikací", "OpenTelemetry Collector Kubernetes"]
---

<article class="blog-article">
<div class="blog-header">
<div class="lp-breadcrumb"><a href="/">VisionOps</a> / Blog</div>
<div class="blog-meta">18. dubna 2026 · 10 min čtení</div>
<h1>OpenTelemetry + Beyla:<br/>monitoring bez změny kódu v roce 2026</h1>
<p class="blog-perex">Největší překážka nasazení observability byl vždy požadavek na instrumentaci kódu. Grafana Beyla mění pravidla hry — eBPF auto-instrumentace přináší traces, metriky a logy z jakékoli aplikace bez jediného řádku změn. Přinášíme praktický návod na kompletní observability stack 2026.</p>
</div>

<div class="blog-content">

## Proč je observability v 2026 jiná

Ještě v roce 2023 bylo nasazení kompletní observability (metriky + logy + traces) komplexní operace vyžadující instrumentaci každé aplikace, správu agentů a integraci desítek komponent. V roce 2026 jsou k dispozici tři technologie, které to dramaticky zjednodušují:

1. **OpenTelemetry** jako vendor-neutral standard pro telemetrii
2. **Grafana Beyla** jako eBPF auto-instrumentace bez změn kódu
3. **Grafana LGTM stack** (Loki, Grafana, Tempo, Mimir) jako integrovaná platforma

Výsledkem je observability stack, který lze nasadit za dny místo měsíců.

## OpenTelemetry Collector: centrální telemetry pipeline

OpenTelemetry Collector je páteří moderního observability stacku. Funguje jako vendor-neutral proxy — přijímá telemetrii ze všech zdrojů, zpracovává ji a posílá do cílových systémů.

### Deployment na Kubernetes

```yaml
apiVersion: opentelemetry.io/v1alpha1
kind: OpenTelemetryCollector
metadata:
  name: otel-collector
spec:
  mode: DaemonSet
  config: |
    receivers:
      otlp:
        protocols:
          grpc:
            endpoint: 0.0.0.0:4317
          http:
            endpoint: 0.0.0.0:4318
      prometheus:
        config:
          scrape_configs:
            - job_name: 'kubernetes-pods'
              kubernetes_sd_configs:
                - role: pod
    processors:
      batch:
        timeout: 1s
      memory_limiter:
        limit_mib: 512
    exporters:
      prometheusremotewrite:
        endpoint: http://prometheus:9090/api/v1/write
      loki:
        endpoint: http://loki:3100/loki/api/v1/push
      otlp/tempo:
        endpoint: http://tempo:4317
    service:
      pipelines:
        metrics:
          receivers: [otlp, prometheus]
          processors: [memory_limiter, batch]
          exporters: [prometheusremotewrite]
        logs:
          receivers: [otlp]
          processors: [batch]
          exporters: [loki]
        traces:
          receivers: [otlp]
          processors: [batch]
          exporters: [otlp/tempo]
```

## Grafana Beyla: eBPF auto-instrumentace

Beyla je revoluce v aplikačním monitoringu. Využívá eBPF (extended Berkeley Packet Filter) pro interceptování síťových volání na úrovni kernelu — bez agenta v aplikaci, bez změn kódu, bez restartu.

### Co Beyla monitoruje automaticky

- **HTTP/HTTPS** — všechny příchozí a odchozí requesty s latencí, status kódy, URL patterny
- **gRPC** — service-to-service komunikace s method-level granularitou
- **SQL** — databázové dotazy s latencí a identifikací slow queries
- **Redis** — cache operace
- **Kafka** — producer/consumer messaging

### Nasazení Beyla na OpenShift

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: beyla
spec:
  selector:
    matchLabels:
      app: beyla
  template:
    spec:
      hostPID: true
      containers:
      - name: beyla
        image: grafana/beyla:latest
        securityContext:
          privileged: true
        env:
        - name: BEYLA_OPEN_PORT
          value: "80,443,8080,8443"
        - name: OTEL_EXPORTER_OTLP_ENDPOINT
          value: "http://otel-collector:4317"
        - name: BEYLA_TRACE_PRINTER
          value: "text"
        volumeMounts:
        - name: host-proc
          mountPath: /host/proc
      volumes:
      - name: host-proc
        hostPath:
          path: /proc
```

**Poznámka pro OpenShift:** Beyla vyžaduje privileged SCC nebo specifická capabilities — `SYS_ADMIN`, `SYS_PTRACE`, `NET_ADMIN`. Na OpenShift je nutné přidat odpovídající SecurityContextConstraints.

## AI-assisted monitoring: od alertů k předvídání

### Grafana Machine Learning

Grafana ML (dostupná v Grafana Cloud i on-premise přes plugin) přidává:

**Anomaly detection** — místo statických threshold alertů (CPU > 80%) se učí normální chování metriky a alertuje na odchylky od baseline. Výsledek: méně false positives, lepší signal-to-noise ratio.

**Forecasting** — predikce budoucích hodnot metrik pro kapacitní plánování. Otázka "kdy nám dojde disk?" má konkrétní odpověď s intervalem spolehlivosti.

### Implementace v Prometheus recording rules

Pro jednodušší AI monitoring bez Grafana ML lze využít Prometheus recording rules s exponential smoothing:

```yaml
groups:
- name: anomaly_detection
  rules:
  - record: job:http_request_duration_seconds:rate5m_avg
    expr: avg_over_time(http_request_duration_seconds_bucket[1h])
  
  - alert: LatencyAnomaly
    expr: |
      (
        rate(http_request_duration_seconds_sum[5m])
        / rate(http_request_duration_seconds_count[5m])
      ) > (
        avg_over_time(
          (rate(http_request_duration_seconds_sum[5m])
          / rate(http_request_duration_seconds_count[5m]))[24h:5m]
        ) * 2
      )
    labels:
      severity: warning
    annotations:
      summary: "Latence je 2× vyšší než 24h průměr"
```

## Kompletní architektura stacku

```
Aplikace (bez změn kódu)
        ↓
  Grafana Beyla (eBPF)
        ↓
OpenTelemetry Collector (DaemonSet)
        ↓
  ┌─────┬──────┬──────┐
  │     │      │      │
Prometheus  Loki  Tempo
  │     │      │      │
  └─────┴──────┴──────┘
        ↓
     Grafana
  (single pane of glass)
        ↓
  Alertmanager → PagerDuty / Slack
```

## Výsledky z produkce (Zentity)

Po nasazení tohoto stacku na OpenShift cluster pro Zentity:

- **Čas do první instrumentace:** 4 hodiny (Beyla DaemonSet deploy + základní dashboardy)
- **Pokrytí aplikací:** 100% — všechny HTTP/gRPC aplikace bez změny kódu
- **Redukce alert fatigue:** ~60% méně false positive alertů oproti statickým thresholdům
- **MTTR (Mean Time to Resolution):** zkráceno průměrně o 40% díky korelaci traces + logů v Grafaně

## Závěr

Observability stack 2026 je signifikantně přístupnější než před dvěma lety. OpenTelemetry jako standard eliminuje vendor lock-in, Beyla odstraňuje nutnost instrumentace kódu a Grafana LGTM stack poskytuje integrovanou platformu pro všechny tři pilíře observability.

Pro Kubernetes a OpenShift prostředí doporučujeme tento stack jako výchozí bod — je open-source, škálovatelný a pokryje 90% potřeb i největších produkčních prostředí.

</div>

<div class="blog-cta">
<h3>Chcete nasadit tento monitoring stack?</h3>
<p>Implementujeme kompletní observability stack na klíč. Ozvěte se.</p>
<a href="mailto:info@visionops.cz" class="cta-button">info@visionops.cz</a>
</div>
</article>
