---
title: "Zentity: OpenShift 4.x na Proxmox clusteru s kompletním observability stackem"
description: "Case study: nasazení Proxmox VE clusteru, Proxmox Backup Server, OpenShift 4.x, ArgoCD GitOps, GitLab CE CI, Ansible provisioning a kompletního OTEL 2026 + Beyla monitoring stacku."
date: 2026-04-18
lastmod: 2026-04-18
keywords: ["OpenShift Proxmox case study", "OpenShift implementace reference", "Proxmox OpenShift produkce", "OpenTelemetry Beyla implementace", "ArgoCD GitOps reference", "DevOps implementace Česká republika"]
---

<section class="lp-hero cs-hero">
<div class="lp-hero-content">
<div class="lp-breadcrumb"><a href="/">VisionOps</a> / Reference / Zentity</div>
<div class="cs-tag">Case Study</div>
<h1>Zentity: Moderní infrastruktura<br/>na klíč pro rok 2026</h1>
<p class="lp-hero-sub">Kompletní transformace infrastruktury — od bare-metal po produkční OpenShift cluster s plnou observabilitou a GitOps workflow.</p>
</div>
</section>

<section class="lp-section">
<div class="lp-container">
<div class="cs-summary-grid">
<div class="cs-summary-item">
<span class="cs-label">Klient</span>
<span class="cs-value">Zentity</span>
</div>
<div class="cs-summary-item">
<span class="cs-label">Oblast</span>
<span class="cs-value">Fintech / IT infrastruktura</span>
</div>
<div class="cs-summary-item">
<span class="cs-label">Platforma</span>
<span class="cs-value">OpenShift 4.x na Proxmox VE</span>
</div>
<div class="cs-summary-item">
<span class="cs-label">Rok</span>
<span class="cs-value">2025–2026</span>
</div>
</div>
</div>
</section>

<section class="lp-section lp-dark">
<div class="lp-container">
<h2>Výchozí situace</h2>
<p>Zentity potřebovala moderní, škálovatelnou a bezpečnou infrastrukturu schopnou provozovat produkční workloady v kontejnerovém prostředí. Požadavky: vysoká dostupnost, plná auditovatelnost nasazení, automatizovaný provisioning a kompletní observabilita infrastruktury i aplikací.</p>
</div>
</section>

<section class="lp-section">
<div class="lp-container">
<h2>Co bylo nasazeno</h2>
<div class="lp-grid-2">

<div class="lp-card">
<h3>1. Proxmox VE cluster + PBS</h3>
<p>Základem byl návrh a implementace Proxmox Virtual Environment clusteru pro virtualizační vrstvu. Součástí bylo:</p>
<ul class="lp-list-sm">
<li>Multi-node Proxmox VE 8.x cluster s Corosync HA</li>
<li>Ceph distributed storage pro shared VM storage</li>
<li>Proxmox Backup Server (PBS) pro deduplikované zálohy</li>
<li>Síťová segmentace přes VLAN a Linux bridges</li>
<li>Automatizovaný provisioning VM přes Ansible</li>
</ul>
</div>

<div class="lp-card">
<h3>2. OpenShift 4.x produkční cluster</h3>
<p>Na Proxmox VE byl nasazen produkční OpenShift cluster v IPI (Installer-Provisioned Infrastructure) konfiguraci:</p>
<ul class="lp-list-sm">
<li>Control plane s vysokou dostupností (3 master nody)</li>
<li>Worker nody s autoscaling konfigurací</li>
<li>OVN-Kubernetes CNI pro síťové politiky</li>
<li>Integrovaný image registry a OAuth provider</li>
<li>TLS certifikáty přes cert-manager a Let's Encrypt</li>
</ul>
</div>

<div class="lp-card">
<h3>3. ArgoCD GitOps</h3>
<p>Veškeré nasazení aplikací a konfigurace clusteru přes GitOps workflow:</p>
<ul class="lp-list-sm">
<li>ArgoCD v HA konfiguraci na OpenShift</li>
<li>App of Apps pattern pro hierarchické řízení aplikací</li>
<li>ApplicationSets pro multi-environment deployment</li>
<li>Automated sync s health check gates</li>
<li>RBAC integrace s OpenShift identity providerem</li>
</ul>
</div>

<div class="lp-card">
<h3>4. GitLab CE + CI pipeline</h3>
<p>Self-hosted GitLab Community Edition jako centrální platforma pro správu kódu a CI/CD:</p>
<ul class="lp-list-sm">
<li>GitLab CE instalace na dedikovaném VM</li>
<li>GitLab CI runnery integrované s OpenShift</li>
<li>Container registry pro build artefakty</li>
<li>Příprava CI pipeline pro automatický build a push do ArgoCD</li>
<li>Branch protection a merge request workflow</li>
</ul>
</div>

<div class="lp-card">
<h3>5. Ansible provisioning</h3>
<p>Veškerý provisioning infrastruktury jako kód přes Ansible:</p>
<ul class="lp-list-sm">
<li>Ansible playbooks pro VM provisioning v Proxmox</li>
<li>Konfigurace OS (hardening, DNS, NTP, sítě)</li>
<li>Idempotentní playbooks pro den-2 operace</li>
<li>Ansible Vault pro správu secrets</li>
<li>Integrace s GitLab CI pro automatické spouštění</li>
</ul>
</div>

<div class="lp-card">
<h3>6. Kompletní observability stack</h3>
<p>Full-stack monitoring pokrývající infrastrukturu, platformu i aplikace:</p>
<ul class="lp-list-sm">
<li>OpenTelemetry Collector jako centrální telemetry pipeline</li>
<li>Grafana Beyla pro eBPF auto-instrumentaci aplikací</li>
<li>Prometheus + Grafana pro metriky a dashboardy</li>
<li>Grafana Loki pro centralizované logování</li>
<li>Grafana Tempo pro distribuované traces</li>
<li>Zabbix pro low-level monitoring infrastruktury</li>
<li>AI-assisted anomaly detection a prediktivní alerting</li>
</ul>
</div>

</div>
</div>
</section>

<section class="lp-section lp-dark">
<div class="lp-container">
<h2>Klíčové technologie</h2>
<div class="tech-badges" style="justify-content: center; display: flex; flex-wrap: wrap; gap: 0.75rem;">
<span>Proxmox VE 8.x</span>
<span>Proxmox Backup Server</span>
<span>OpenShift 4.x</span>
<span>ArgoCD</span>
<span>GitLab CE</span>
<span>Ansible</span>
<span>OpenTelemetry</span>
<span>Grafana Beyla</span>
<span>Prometheus</span>
<span>Grafana</span>
<span>Loki</span>
<span>Tempo</span>
<span>Zabbix</span>
<span>Ceph</span>
<span>Helm</span>
<span>cert-manager</span>
</div>
</div>
</section>

<section class="lp-section">
<div class="lp-container">
<h2>Výsledky</h2>
<div class="lp-grid-3">
<div class="lp-card cs-result">
<h3>Plná automatizace</h3>
<p>Od provisioning VM přes deployment aplikace po monitoring — vše automatizované. Žádné ruční kroky v produkci.</p>
</div>
<div class="lp-card cs-result">
<h3>Kompletní viditelnost</h3>
<p>Metriky, logy a traces z každé části stacku korelované v jednom místě — Grafana jako single pane of glass.</p>
</div>
<div class="lp-card cs-result">
<h3>GitOps jako standard</h3>
<p>Každá změna v infrastruktuře i aplikacích prochází Git review. Auditovatelnost, rollback a compliance ready.</p>
</div>
</div>
</div>
</section>

<section class="lp-cta">
<div class="lp-container">
<h2>Chcete podobné řešení?</h2>
<p>Probereme vaši situaci a navrhneme architekturu šitou na míru.</p>
<a href="mailto:info@visionops.cz" class="cta-button-large">info@visionops.cz</a>
</div>
</section>
