# Déploiement de microservices NGINX avec Ingress HTTPS

Ce projet déploie deux microservices NGINX dans un cluster Kubernetes local (Kind), exposés en HTTPS via Ingress NGINX et des certificats TLS auto-signés.

---

## Objectifs

- Créer un cluster Kubernetes local avec **Kind**
- Déployer 2 microservices :
  - `hello-risf` : image Docker personnalisée
  - `hello-itsf` : image NGINX officielle + PV/PVC
- Exposer les deux services via **Ingress** en **HTTPS**
- Utiliser une **PKI locale** pour signer les certificats TLS
- S'assurer que les **conteneurs ne tournent pas en root**

---

## Structure du projet

```
devops-test/
├── kind-config.yaml
├── hello-risf/
│   ├── index.html
│   ├── Dockerfile
│   ├── risf-nginx.yaml
│   ├── risf-ingress.yaml
│   ├── risf.key / risf.crt
├── hello-itsf/
│   ├── index.html
│   ├── nginx.conf
│   ├── itsf-nginx.yaml
│   ├── itsf-ingress.yaml
│   ├── itsf-pv.yaml
│   ├── itsf.key / itsf.crt
```

---

## Prérequis

- Docker Desktop (Mac M1)
- `kubectl` et `kind` installés
- Ajout de noms locaux dans `/etc/hosts` :
  ```
  127.0.0.1 hello-risf.local.domain
  127.0.0.1 hello-itsf.local.domain
  ```

---

## Déploiement complet

### 1. Créer le cluster

```bash
kind create cluster --config kind-config.yaml
```

### 2. Construire et charger l’image Docker RISF

```bash
cd hello-risf
docker build -t risf-nginx .
kind load docker-image risf-nginx --name risf-cluster
```

### 3. Déployer les microservices et certificats

```bash
# hello-risf
kubectl apply -f risf-nginx.yaml
kubectl create secret tls risf-tls --cert=risf.crt --key=risf.key
kubectl apply -f risf-ingress.yaml

# hello-itsf
kubectl apply -f ../hello-itsf/itsf-pv.yaml
kubectl apply -f ../hello-itsf/itsf-nginx.yaml
kubectl create secret tls itsf-tls --cert=itsf.crt --key=itsf.key
kubectl apply -f ../hello-itsf/itsf-ingress.yaml
```

### 4. Installer l’Ingress Controller

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.0/deploy/static/provider/kind/deploy.yaml

# puis
kubectl label node risf-cluster-control-plane ingress-ready=true
```

---

## Accès HTTPS

Ouvrir dans le navigateur :

- 🔒 https://hello-risf.local.domain
- 🔒 https://hello-itsf.local.domain

Ignore l’avertissement de certificat (car certificat auto-signé).

---

## Contraintes respectées

| Critère                                   | Statut |
|-------------------------------------------|--------|
| Déploiement local Kubernetes avec Kind     | ✅     |
| Image Docker custom (hello-risf)           | ✅     |
| Volume local monté pour ITSF               | ✅     |
| Utilisateur non-root dans les pods         | ✅     |
| Ingress avec HTTPS                         | ✅     |
| Certificats TLS signés avec une CA locale  | ✅     |

---

## Auteur

Réalisé par **Mitra Dadgar** – [github.com/mitra1367](https://github.com/mitra1367)
