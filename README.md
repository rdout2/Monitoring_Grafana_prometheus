# 📈 Monitoring avec Prometheus, Node Exporter et Grafana (via Docker Compose)

Ce guide vous montre comment mettre en place un stack de monitoring complet avec **Prometheus**, **Node Exporter** et **Grafana**, via Docker Compose. Vous apprendrez à créer vos dashboards, utiliser des variables et importer des dashboards existants.

---

## 🧰 Prérequis

- Docker & Docker Compose installés
- Connaissances de base sur Prometheus & les métriques système

---

## ⚙️ Étape 1 : Configuration de Prometheus, Node Exporter & Grafana

### Fichier `compose.yaml`

```yaml
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    restart: unless-stopped
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    ports:
      - 9090:9090

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    restart: unless-stopped
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - --path.procfs=/host/proc
      - --path.rootfs=/rootfs
      - --path.sysfs=/host/sys
      - --collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)
    ports:
      - 9100:9100

  grafana:
    image: grafana/grafana-oss:latest
    container_name: grafana
    ports:
      - 3000:3000
    volumes:
      - grafana_data:/var/lib/grafana
    depends_on:
      - prometheus

networks:
  default:
    name: prometheus-grafana
    driver: bridge

volumes:
  prometheus_data:
  grafana_data:
```

### Fichier `prometheus.yml`

```yaml
global:
  scrape_interval: 10s

scrape_configs:
  - job_name: node-exporter
    static_configs:
      - targets:
          - 'node-exporter:9100'
```

### Lancer la stack

```bash
docker compose up -d
```

Accéder aux cibles : http://localhost:9090/targets

---

## 📊 Accéder à Grafana

- Interface : http://localhost:3000
- Identifiants par défaut : `admin` / `admin`

---

## 🔗 Connecter Prometheus à Grafana

1. Aller dans **Connections > Add new connection**
2. Choisir **Prometheus**
3. URL : `http://prometheus:9090`
4. Cliquez sur **Save & test**

---

## 📈 Créer des Dashboards

### ▶️ Uptime (Stat)

```promql
node_time_seconds{instance="node-exporter:9100"} - node_boot_time_seconds{instance="node-exporter:9100"}
```

- Type : Stat  
- Unité : seconds

---

### 🧠 RAM utilisée (Time series)

```promql
node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes
```

- Type : Time series  
- Unité : bytes (IEC)

---

### 💽 Espace disque utilisé (Gauge)

```promql
1 - (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"})
```

- Type : Gauge  
- Unité : Percent (0–1)  
- Ajoutez des seuils (thresholds)

---

## 🧩 Ajouter des variables dans Grafana

1. Aller dans **Dashboard Settings > Variables**
2. Ajouter une variable :
   - Name : `job`
   - Type : `Query`
   - Data source : `Prometheus`
   - Query : `label_values(node_uname_info, job)`
3. Cliquez sur **Update**

Utilisation dans vos requêtes :

```promql
{job="$job"}
```

---

## 📥 Importer un Dashboard existant

1. Aller dans **Dashboards > New > Import**
2. Coller l’ID ou l’URL :

```
1860
https://grafana.com/grafana/dashboards/1860-node-exporter-full/
```

3. Sélectionner la source Prometheus
4. Importer 🎉

---

## ✅ Résultat

🎉 Vous disposez maintenant d’un environnement de monitoring complet avec Prometheus, Node Exporter et Grafana.

---

## 🔭 Aller plus loin

- Ajouter des alertes Prometheus
- Monitorer plusieurs machines (ajouter plusieurs `node-exporter` avec des labels différents)
- Ajouter des notifications dans Grafana (Slack, Discord, Email…)

---

> Ce guide est open-source et réutilisable. Contributions bienvenues 🚀
