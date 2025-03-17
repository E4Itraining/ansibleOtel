# OpenTelemetry Ansible Playbook

##  Description

Ce projet Ansible permet de **déployer un OpenTelemetry Collector avec 
une intégration avancée aux systèmes de monitoring et logging :  
- Elasticsearch, Logstash, Kibana (ELK) 
- Dynatrace pour l'observabilité avancée  
- APM Server pour l'analyse des traces 
- Loki, Kafka, et S3 pour la gestion des logs 
- Filtrage avancé des logs et traces  
- Optimisation des performances avec Tail Sampling  

##  Structure du projet

```
otel-ansible/
├── inventory.ini                # Fichier d'inventaire définissant les hôtes
├── playbook.yml                 # Playbook Ansible principal
├── argo-opentelemetry.yaml       # Déploiement GitOps via ArgoCD
├── files/
│   └── kibana_apm_dashboard.ndjson  # Dashboard préconfiguré pour Kibana APM
└── roles/
    ├── otel_collector/          # Déploiement OpenTelemetry Collector
    │   ├── tasks/
    │   │   └── main.yml
    │   ├── templates/
    │   │   └── otel-collector-config.yaml.j2
    │   └── handlers/
    │       └── main.yml
    ├── elasticsearch/           # Déploiement et configuration Elasticsearch
    ├── logstash/                # Déploiement et configuration Logstash
    ├── s3_storage/              # Configuration de la sauvegarde des logs sur S3
    └── apm_server/              # Déploiement APM Server
```

##  Déploiement

### 1️) Configurer l’inventaire Ansible**
Modifiez `inventory.ini` pour définir les hôtes cibles.

### 2) Lancer le Playbook
Exécutez la commande suivante pour déployer OpenTelemetry et ELK :
```bash
ansible-playbook -i inventory.ini playbook.yml
```

##  Configuration OpenTelemetry

Le fichier de configuration **`otel-collector-config.yaml.j2`** est basé sur les paramètres suivants :  
- Receveurs : OTLP (gRPC, HTTP), Filelog  
- Processeurs : Batch, Attributs, Filtrage, Tail Sampling  
- Exportateurs : 
  - Elasticsearch (`https://elastic:9200`)  
  - APM Server (`https://apm-server.mydomain.com:8200`)  
  - Dynatrace (`https://dynatrace-instance/api/v2/otlp`)  
  - Loki (`http://loki:3100`)  
  - Kafka (`kafka:9092`)  

## ⚙️ Configuration avancée

### Filtrage des Traces**
Les traces peuvent être filtrées en fonction de l’environnement ou du profil utilisateur :
```yaml
processors:
  filter:
    traces:
      exclude:
        match_type: strict
        attributes:
          user.role: ["guest"]
          service.environment: ["dev"]
```

### Optimisation Tail Sampling**
Permet de limiter le volume des traces collectées :
```yaml
processors:
  tailsampling:
    policies:
      - type: rate_limiting
        rate_limiting: { spans_per_second: 100 }
      - type: probabilistic
        probabilistic: { sampling_percentage: 50 }
```

### Sécurisation avec API Key Dynatrace**
```yaml
exporters:
  dynatrace:
    endpoint: "https://dynatrace-instance/api/v2/otlp"
    api_key: "{{ dynatrace_api_key }}"
```

##  Visualisation et Analyse

- Kibana APM permet d’afficher les traces et logs OpenTelemetry  
- Dynatrace offre une analyse avancée des performances  
- Loki + Grafana (anticipation): facilite la visualisation des logs bruts  




