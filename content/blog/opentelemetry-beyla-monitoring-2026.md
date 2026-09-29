---
title: "OpenTelemetry + Beyla: monitoring bez změny kódu"
description: "Jak nasadit kompletní observability stack v roce 2026 s OpenTelemetry Collector, Grafana Beyla eBPF auto-instrumentací a AI-assisted monitoringem na Kubernetes a OpenShift."
date: 2026-04-18
breadcrumb: "OpenTelemetry a Beyla"
keywords: ["opentelemetry", "beyla", "ebpf", "observability", "monitoring", "openshift", "kubernetes"]
---

<article class="blog-article">
<div class="blog-header">
{{< breadcrumb >}}
<div class="blog-meta">18. dubna 2026 · 10 min čtení · ověřeno podle aktuální dokumentace 29. září 2026</div>
<h1>OpenTelemetry + Beyla:<br/>monitoring bez změny kódu v roce 2026</h1>
<p class="blog-perex">Největší překážka nasazení observability byl vždy požadavek na instrumentaci kódu. Grafana Beyla to mění — eBPF auto-instrumentace přináší traces a RED metriky z jakékoli aplikace bez jediného řádku změn. Přinášíme praktický návod na kompletní observability stack 2026.</p>
</div>

<div class="blog-content">

## Proč je observability v 2026 jiná

Ještě v roce 2023 bylo nasazení kompletní observability (metriky + logy + traces) komplexní operace vyžadující instrumentaci každé aplikace, správu agentů a integraci desítek komponent. V roce 2026 jsou k dispozici tři technologie, které to dramaticky zjednodušují:

1. **OpenTelemetry** jako vendor-neutral standard pro telemetrii
2. **Grafana Beyla** jako eBPF auto-instrumentace bez změn kódu
3. **Grafana LGTM stack** (Loki, Grafana, Tempo, Mimir) jako integrovaná platforma

Výsledkem je observability stack, který lze nasadit výrazně rychleji než dřív.

## OpenTelemetry Collector: centrální telemetry pipeline

OpenTelemetry Collector je páteří moderního observability stacku. Funguje jako vendor-neutral proxy — přijímá telemetrii ze všech zdrojů, zpracovává ji a posílá do cílových systémů.

### Deployment na Kubernetes

```yaml
apiVersion: opentelemetry.io/v1beta1
kind: OpenTelemetryCollector
metadata:
  name: otel-collector
spec:
  mode: deployment
  config:
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
        check_interval: 1s
        limit_mib: 512
    exporters:
      prometheusremotewrite:
        endpoint: http://prometheus:9090/api/v1/write
      otlp_http/loki:
        endpoint: http://loki:3100/otlp
      otlp_grpc/tempo:
        endpoint: tempo:4317
        tls:
          insecure: true
    service:
      pipelines:
        metrics:
          receivers: [otlp, prometheus]
          processors: [memory_limiter, batch]
          exporters: [prometheusremotewrite]
        logs:
          receivers: [otlp]
          processors: [batch]
          exporters: [otlp_http/loki]
        traces:
          receivers: [otlp]
          processors: [batch]
          exporters: [otlp_grpc/tempo]
```

Collector běží jako Deployment, aby Prometheus receiver nescrapoval stejné pody z každého nodu. Operator k němu vytvoří Service `otel-collector-collector`. Prometheus receiver s `kubernetes_sd_configs` potřebuje ServiceAccount s právy list/watch na pody a Prometheus musí mít zapnutý příjem remote write (`--web.enable-remote-write-receiver`).

## Grafana Beyla: eBPF auto-instrumentace

Beyla výrazně mění aplikační monitoring. Využívá eBPF (extended Berkeley Packet Filter) — sondy na úrovni kernelu i uprobes v aplikačních binárkách — bez agenta v aplikaci, bez změn kódu, bez restartu. Vyžaduje Linux kernel 5.8+ s BTF, případně RHEL 8 a odvozené distribuce s backportovaným eBPF.

Jádro Beyly Grafana v roce 2025 darovala projektu OpenTelemetry, kde pokračuje jako OpenTelemetry eBPF Instrumentation (OBI). Beyla zůstává distribucí OBI od Grafany a obě používají stejné YAML schéma konfigurace; proměnné prostředí OBI ale mají prefix `OTEL_EBPF_` místo `BEYLA_`.

### Co Beyla monitoruje automaticky

- **HTTP/HTTPS** — všechny příchozí a odchozí requesty s latencí, status kódy, URL patterny
- **gRPC** — service-to-service komunikace s method-level granularitou
- **SQL** — databázové dotazy (PostgreSQL, MySQL, MSSQL) s latencí
- **Redis** — cache operace
- **Kafka** — producer/consumer messaging

### Nasazení Beyla na OpenShift

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: beyla
  labels:
    app: beyla
spec:
  selector:
    matchLabels:
      app: beyla
  template:
    metadata:
      labels:
        app: beyla
    spec:
      serviceAccountName: beyla
      hostPID: true
      containers:
      - name: beyla
        image: grafana/beyla:latest
        env:
        - name: BEYLA_AUTO_TARGET_EXE
          value: "*/my-app"
        - name: OTEL_EXPORTER_OTLP_ENDPOINT
          value: "http://otel-collector-collector:4317"
        - name: BEYLA_KUBE_METADATA_ENABLE
          value: "true"
        securityContext:
          runAsUser: 0
          readOnlyRootFilesystem: true
          capabilities:
            add: [BPF, SYS_PTRACE, NET_RAW, CHECKPOINT_RESTORE, DAC_READ_SEARCH, PERFMON, SYS_ADMIN]
            drop: [ALL]
        volumeMounts:
        - name: var-run-beyla
          mountPath: /var/run/beyla
        - name: cgroup
          mountPath: /sys/fs/cgroup
        - name: tracefs
          mountPath: /sys/kernel/tracing
      volumes:
      - name: var-run-beyla
        emptyDir: {}
      - name: cgroup
        hostPath:
          path: /sys/fs/cgroup
      - name: tracefs
        hostPath:
          path: /sys/kernel/tracing
```

V DaemonSet režimu se aplikace vybírají podle názvu spustitelného souboru (`BEYLA_AUTO_TARGET_EXE`), protože porty jsou uvnitř podů. Kubernetes metadata (`BEYLA_KUBE_METADATA_ENABLE`) vyžadují ServiceAccount `beyla` s právy číst pody a další objekty clusteru.

**Poznámka pro OpenShift:** výchozí SCC neumožňuje přístup k hostiteli, UID 0 ani potřebné capabilities. Podle dokumentace Beyly vytvořte vlastní SCC s `allowHostPID`, `allowHostDirVolumePlugin`, `runAsUser: MustRunAs` (UID 0), capabilities výše plus `NET_ADMIN` a `allowPrivilegedContainer: false`, a přidělte ji jen ServiceAccountu Beyly: `oc adm policy add-scc-to-user beyla -z beyla -n <namespace>`.

## AI-assisted monitoring: od alertů k předvídání

### Grafana Cloud Machine Learning

Machine learning v Grafana Cloud přidává:

**Anomaly detection** — místo statických threshold alertů (CPU > 80%) se učí normální chování metriky a alertuje na odchylky od baseline. Výsledek: méně false positives, lepší signal-to-noise ratio.

**Forecasting** — predikce budoucích hodnot metrik pro kapacitní plánování. Otázka "kdy nám dojde disk?" má konkrétní odpověď s intervalem predikce.

### Implementace v Prometheus recording rules

Bez Grafana Cloud lze jednoduchou detekci odchylek postavit na Prometheus recording rule a porovnání s klouzavým průměrem za 24 hodin:

```yaml
groups:
- name: anomaly_detection
  rules:
  - record: job:http_request_duration_seconds:mean5m
    expr: |
      sum by (job) (rate(http_request_duration_seconds_sum[5m]))
      / sum by (job) (rate(http_request_duration_seconds_count[5m]))

  - alert: LatencyAnomaly
    expr: |
      job:http_request_duration_seconds:mean5m
      > avg_over_time(job:http_request_duration_seconds:mean5m[24h]) * 2
    labels:
      severity: warning
    annotations:
      summary: "Latence je 2× vyšší než 24h průměr"
```

## Kompletní architektura stacku

```
Aplikace (bez změn kódu)
        ↓
  Grafana Beyla (eBPF)  +  logy aplikací (OTLP)
        ↓
OpenTelemetry Collector (Deployment)
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

## Nasazení v praxi (Zentity)

Tento stack provozujeme na [OpenShift clusteru pro Zentity](/reference/zentity-openshift-proxmox/). Beyla instrumentuje HTTP a gRPC aplikace bez změny kódu a traces jsou v Grafaně korelované s logy a metrikami.

## Závěr

Observability stack 2026 je výrazně přístupnější než před dvěma lety. OpenTelemetry jako standard eliminuje vendor lock-in, Beyla odstraňuje nutnost instrumentace kódu a Grafana LGTM stack poskytuje integrovanou platformu pro všechny tři pilíře observability.

Pro Kubernetes a OpenShift prostředí doporučujeme tento stack jako výchozí bod — je open-source, škálovatelný a pokryje většinu běžných potřeb produkčních prostředí. Co zahrnuje naše [nasazení monitoringu a observability](/sluzby/monitoring-observability/), najdete na stránce služby.

</div>

<div class="blog-cta">
<h3>Chcete nasadit tento monitoring stack?</h3>
<p>Implementujeme kompletní observability stack na klíč. Ozvěte se.</p>
<a href="mailto:info@visionops.cz" class="cta-button">info@visionops.cz</a>
</div>
</article>
