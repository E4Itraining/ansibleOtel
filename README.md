# OpenTelemetry Ansible Playbook

## Description

Ce projet Ansible permet de **déployer un OpenTelemetry Collector** avec **intégration avancée** aux systèmes de monitoring et logging :  
- **Elasticsearch (par défaut) et Dynatrace (en second) avec routage dynamique**  
- **APM Server pour l'analyse des traces**  
- **Loki, Kafka, et Azure Blob Storage pour la gestion des logs**  
- **Filtrage avancé des logs et traces**  
- **Optimisation des performances avec Tail Sampling**  

##  Structure du projet

```
otel-ansible/
├── inventory.ini                # Fichier d'inventaire définissant les hôtes
├── playbook.yml                 # Playbook Ansible principal
└── roles/
    ├── otel_collector/          # Déploiement OpenTelemetry Collector
    ├── elasticsearch/           # Déploiement et configuration Elasticsearch
    ├── logstash/                # Déploiement et configuration Logstash
    ├── azure_storage/           # Configuration de la sauvegarde des logs sur Azure Blob Storage
    ├── apm_server/              # Déploiement APM Server
```

## 🚀 Déploiement

### 1 **Configurer l’inventaire Ansible**
Modifiez `inventory.ini` pour définir les hôtes cibles.

### 2 **Lancer le Playbook**
Exécutez la commande suivante pour déployer OpenTelemetry et ELK :
```bash
ansible-playbook -i inventory.ini playbook.yml
```

##  Configuration avancée

### Routage Dynamique des Traces**
Par défaut, les traces sont envoyées à **Elasticsearch**, mais pour les projets spécifiques (ex: `wbk`), elles sont redirigées vers **Dynatrace**.
```yaml
connectors:
  routing:
    default_exporters: [elasticsearch]
    table:
      - statement: 'attributes["project_name"] == "wbk"'
        exporters: [dynatrace]
      - statement: 'true'
        exporters: [elasticsearch]
```

### Filtrage des Traces
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

### Optimisation Tail Sampling
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

##  Visualisation et Analyse

- **Kibana APM** permet d’afficher les traces et logs OpenTelemetry  
- **Dynatrace** offre une analyse avancée des performances (uniquement pour `wbk`)  
- **Loki + Grafana** facilite la visualisation des logs bruts  
- **Azure Blob Storage** permet une **conservation longue durée des logs**  




