# Déploiement de l'application quartify

Ce dépôt permet le déploiement de l'application [quartify](https://github.com/ddotta/quartify) sur un cluster Kubernetes. La configuration présentée est spécifiquement adaptée au [SSP Cloud](https://datalab.sspcloud.fr/home).

## Description

quartify est une application Shiny qui permet de convertir des scripts R en documents Quarto (.qmd). Cette application nécessite Quarto pour le rendu HTML, c'est pourquoi le déploiement sur SSP Cloud est idéal.

## Prérequis

- Un compte sur [SSP Cloud](https://datalab.sspcloud.fr/)
- Kubectl configuré (optionnel)
- Helm 3+ (optionnel)

## Déploiement rapide via SSP Cloud

### Option 1 : Interface Web SSP Cloud

1. Se connecter sur [SSP Cloud](https://datalab.sspcloud.fr/)
2. Aller dans "Catalog" → "Helm Charts"
3. Sélectionner "Custom Helm Chart"
4. Configuration :
   - **Name** : quartify
   - **Chart Repository** : `https://github.com/ddotta/quartify_deployment`
   - **Values File** : `values-sspcloud.yaml`
5. Lancer le service

### Option 2 : Helm CLI

```bash
# Cloner le dépôt
git clone https://github.com/ddotta/quartify_deployment.git
cd quartify_deployment

# Installer avec Helm
helm install quartify . -f values-sspcloud.yaml

# Mettre à jour
helm upgrade quartify . -f values-sspcloud.yaml

# Désinstaller
helm uninstall quartify
```

## Configuration

### Fichiers principaux

- **Chart.yaml** : Métadonnées du chart Helm
- **values.yaml** : Configuration par défaut
- **values-sspcloud.yaml** : Configuration spécifique SSP Cloud
- **templates/** : Templates Kubernetes (Deployment, Service, Ingress)

### Personnalisation

Éditez `values-sspcloud.yaml` pour personnaliser :

```yaml
# Ressources
resources:
  requests:
    cpu: 1000m      # Ajuster selon besoin
    memory: 2Gi     # Ajuster selon besoin

# Domaine
ingress:
  hosts:
    - host: votre-quartify.lab.sspcloud.fr  # Changer le sous-domaine
```

## Architecture

```
┌─────────────────────────────────────────┐
│           Internet/SSP Cloud            │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│         Ingress (HTTPS/TLS)             │
│    quartify.lab.sspcloud.fr             │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│      Service (ClusterIP:80)             │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│      Deployment (Pod)                   │
│  ┌─────────────────────────────────┐   │
│  │ Init Container                  │   │
│  │ - Install Quarto 1.4.549        │   │
│  │ - Install R dependencies        │   │
│  └─────────────────────────────────┘   │
│  ┌─────────────────────────────────┐   │
│  │ Main Container                  │   │
│  │ - Image: onyxia-r-minimal:4.3.3 │   │
│  │ - Shiny App on port 3838        │   │
│  │ - quartify from GitHub          │   │
│  └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
```

## Ressources

- **CPU** : 1 vCPU (request), 2 vCPU (limit)
- **Mémoire** : 2 GB (request), 4 GB (limit)
- **Port** : 3838 (Shiny)

## Fonctionnalités

✅ Conversion R → Quarto (.qmd)  
✅ Rendu HTML complet avec Quarto  
✅ Interface Shiny interactive  
✅ Support des numéros de ligne  
✅ HTTPS/TLS automatique  
✅ Hébergement gratuit sur SSP Cloud  

## Dépannage

### Le service ne démarre pas

```bash
# Vérifier les pods
kubectl get pods -l app.kubernetes.io/name=quartify

# Voir les logs
kubectl logs -l app.kubernetes.io/name=quartify

# Décrire le pod
kubectl describe pod -l app.kubernetes.io/name=quartify
```

### Quarto non trouvé

Vérifier que l'init container s'est bien exécuté :

```bash
kubectl logs <pod-name> -c install-quarto
```

### Problèmes de ressources

Augmenter les limites dans `values-sspcloud.yaml` :

```yaml
resources:
  limits:
    memory: 6Gi  # Au lieu de 4Gi
```

## Mise à jour

### Mettre à jour la version de quartify

Le déploiement récupère automatiquement la dernière version depuis GitHub. Pour forcer une mise à jour :

```bash
# Supprimer le pod pour forcer un redéploiement
kubectl delete pod -l app.kubernetes.io/name=quartify

# Ou via Helm
helm upgrade quartify . -f values-sspcloud.yaml
```

### Mettre à jour Quarto

Modifier la version dans `values.yaml` :

```yaml
initContainers:
  - name: install-quarto
    command:
      - /bin/bash
      - -c
      - |
        wget https://github.com/quarto-dev/quarto-cli/releases/download/v1.5.0/quarto-1.5.0-linux-amd64.deb
        # ...
```

## Support

- **Issues quartify** : https://github.com/ddotta/quartify/issues
- **Issues déploiement** : https://github.com/ddotta/quartify_deployment/issues
- **Documentation SSP Cloud** : https://docs.sspcloud.fr/

## Licence

Ce dépôt de déploiement est sous licence MIT. L'application quartify a sa propre licence.
