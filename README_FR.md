# Déploiement de Quartify

 [English version](README.md)

Configuration Helm/Kubernetes pour déployer l'application web Quartify sur le SSP Cloud (Onyxia).

## Description

Ce dépôt contient la configuration de déploiement pour [Quartify](https://github.com/ddotta/quartify), un package R qui convertit automatiquement des scripts R en documents Quarto markdown (.qmd).

**Image Docker** : [ddottaagr/quartify:latest](https://hub.docker.com/r/ddottaagr/quartify)

## Fonctionnalités de l'application

L'application web Quartify offre :

-  Interface Shiny interactive (EN/FR)
-  Conversion R  Quarto (.qmd)
-  **Rendu HTML avec Quarto** (Quarto est installé dans l'image Docker)
-  Support des callouts, tabsets, diagrammes Mermaid
-  Numéros de ligne source pour la traçabilité
-  Thèmes Quarto personnalisables (25+ thèmes)
-  Téléchargement des fichiers .qmd et .html générés

## Déploiement sur SSP Cloud

### Prérequis

- Accès au [SSP Cloud](https://datalab.sspcloud.fr/)
- Compte utilisateur actif

### Option 1 : Via l'interface web (Recommandé)

1. Connectez-vous sur https://datalab.sspcloud.fr/
2. Allez dans **Mon Labo**  **Catalogue de services**
3. Recherchez \"Shiny\" ou créez un service personnalisé
4. Dans la configuration :
   - **Repository** : `https://github.com/ddotta/quartify_deployment`
   - **Chart** : `.` (racine du dépôt)
   - **Values file** : `values-sspcloud.yaml`
5. Cliquez sur **Lancer**
6. Une fois le service démarré, cliquez sur l'URL fournie

### Option 2 : Via Helm CLI

```bash
# Cloner le dépôt
git clone https://github.com/ddotta/quartify_deployment.git
cd quartify_deployment

# Installer avec Helm
helm install quartify . -f values-sspcloud.yaml

# Obtenir l'URL du service
kubectl get ingress
```

### Option 3 : Via kubectl

```bash
# Appliquer directement les manifests
kubectl apply -f templates/
```

## Configuration

### Fichiers principaux

- **`Chart.yaml`** : Métadonnées du chart Helm
- **`values.yaml`** : Configuration par défaut
- **`values-sspcloud.yaml`** : Configuration spécifique SSP Cloud
- **`templates/`** : Manifests Kubernetes (Deployment, Service, Ingress)

### Personnalisation

Pour modifier la configuration, éditez `values-sspcloud.yaml` :

```yaml
image:
  repository: ddottaagr/quartify
  tag: latest
  pullPolicy: Always

resources:
  limits:
    memory: \"2Gi\"
    cpu: \"2000m\"
  requests:
    memory: \"1Gi\"
    cpu: \"1000m\"

service:
  type: ClusterIP
  port: 3838

ingress:
  enabled: true
  hosts:
    - host: quartify.lab.sspcloud.fr
```

## Structure du dépôt

```
quartify_deployment/
 Chart.yaml                 # Métadonnées Helm
 values.yaml                # Configuration par défaut
 values-sspcloud.yaml       # Configuration SSP Cloud
 templates/
    deployment.yaml        # Déploiement Kubernetes
    service.yaml          # Service Kubernetes
    ingress.yaml          # Ingress pour l'exposition
    _helpers.tpl          # Templates Helm helpers
 README.md                  # Version anglaise
 README_FR.md              # Ce fichier
```

## Image Docker

L'image Docker `ddottaagr/quartify` contient :

- **R 4.4.1** (rocker/shiny)
- **Shiny Server**
- **Quarto 1.4.549**  (permet le rendu HTML)
- **Package quartify** (installé depuis GitHub)
- Toutes les dépendances R nécessaires

### Construction de l'image

L'image est construite automatiquement via GitHub Actions à chaque push sur la branche `main` du dépôt [quartify](https://github.com/ddotta/quartify).

Pour construire manuellement :

```bash
# Cloner le dépôt principal
git clone https://github.com/ddotta/quartify.git
cd quartify

# Construire l'image
docker build -t ddottaagr/quartify:latest .

# Tester localement
docker run -p 3838:3838 ddottaagr/quartify:latest
```

## Dépannage

### Le service ne démarre pas

```bash
# Vérifier les logs du pod
kubectl get pods
kubectl logs <pod-name>

# Vérifier les événements
kubectl describe pod <pod-name>
```

### Problèmes de ressources

Si le pod est en état `OOMKilled` (manque de mémoire), augmentez les limites dans `values-sspcloud.yaml` :

```yaml
resources:
  limits:
    memory: \"4Gi\"  # Augmenter à 4Go
```

### L'application ne répond pas

Vérifiez que le port est correct :

```bash
# Le service doit être sur le port 3838
kubectl get service quartify
```

## Mise à jour

Pour mettre à jour l'application avec une nouvelle version de l'image :

```bash
# Mettre à jour le déploiement
helm upgrade quartify . -f values-sspcloud.yaml

# Ou forcer le redémarrage
kubectl rollout restart deployment/quartify
```

## Liens utiles

-  **Dépôt principal** : https://github.com/ddotta/quartify
-  **Image Docker** : https://hub.docker.com/r/ddottaagr/quartify
-  **Documentation** : https://ddotta.github.io/quartify/
-  **SSP Cloud** : https://datalab.sspcloud.fr/

## Support

Pour toute question ou problème :

- Ouvrir une [issue sur GitHub](https://github.com/ddotta/quartify/issues)
- Consulter la [documentation complète](https://ddotta.github.io/quartify/)

## Licence

Ce projet de déploiement suit la même licence que le projet principal Quartify (MIT).
